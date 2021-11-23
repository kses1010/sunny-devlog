---
title: 'AWS CodeDeploy, ELB(L7)을 이용한 무중단 배포 - 1/2'
date: 2021-11-23
category: 'AWS'
draft: false
---

참고사항입니다.

2021년 3월 초기에 작성했던 문서라 현재 AWS 콘솔과는 달라질 수 있다는점 꼭 확인해주세요!


# 무중단 배포 아키텍처

![image](https://user-images.githubusercontent.com/49144662/142972947-cf9f134e-6b7f-43c1-91fb-b7cba3b3eb8f.png)

# 0-1. AWS IAM CodeDeploy 역할설정하기

IAM에서 CodeDeploy Role을 생성합니다.

![image](https://user-images.githubusercontent.com/49144662/142973033-33759a75-6371-4193-9bd6-291db280401c.png)

CodeDeploy Role 이름을 지정하고 저장합니다.

# 0-2. AWS CodeDeploy 설치

원본 인스턴스에서 CodeDeploy를 설치하고 AMI 이미지를 따면 훨씬 편합니다.

### **Ubuntu 18.04 에서 Directory not empty @ dir_s\*_*rmdir 에러**

- CodeDeploy agent는 Ruby로 작동하기 떄문에 ruby 설치는 필수!
- [Ubuntu 18.04](http://releases.ubuntu.com/18.04/) 에서는 배포된 파일들을 보관할 폴더가 비어있지 않다면 `Directory not empty @ dir_s_rmdir` 에러가 발생합니다.
- ruby 버전이 2.5 이상부터는 ruby 문법 때문에 해당 에러가 발생한다고 합니다. 루비 버전을 꼭 확인!!

```bash
sudo apt remove --purge ruby
sudo apt autoremove
sudo apt -y install software-properties-common
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt update
sudo apt install -y ruby2.4
ruby --version
ruby 2.4.6p354 (2019-04-01 revision 67394) [x86_64-linux-gnu]
```

자바추가 설치

```bash
# corretto 8 설치
wget -O- <https://apt.corretto.aws/corretto.key> | sudo apt-key add - 
 sudo add-apt-repository 'deb <https://apt.corretto.aws> stable main'

sudo apt-get update
sudo apt-get install -y java-1.8.0-amazon-corretto-jdk

# CodeDeploy 설치 및 실행
cd /home/ubuntu
wget <https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install>
chmod +x ./install
sudo ./install auto

# codeDeploy 확인
sudo service codedeploy-agent status
# 실행
sudo service codedeploy-agent start
```

# 1. EC2 AMI 설정하기

![image](https://user-images.githubusercontent.com/49144662/142973054-b1eb1a84-faae-4d2c-99c5-4f3ed7c65d27.png)

Auto Scaling을 설정하려면 EC2 AMI가 필요합니다. 먼저 deploy할 EC2를 선택하여 AMI를 생성합니다.

![image](https://user-images.githubusercontent.com/49144662/142973066-2cf6dd63-325d-4f85-a3ba-6a3a847c05c3.png)

![image](https://user-images.githubusercontent.com/49144662/142973106-4dd68dc7-313a-4e0d-93de-3fe0b67318ee.png)

재부팅 안함을 활성화 하여 원본 인스턴스를 재부팅을 막을 수 있습니다. 만약 선택하지 않으면 인스턴스가 재부팅하여 서버가 꺼지기 때문에 꼭 확인하셔야 합니다.

# 2. ELB 생성하기

![image](https://user-images.githubusercontent.com/49144662/142973129-656515bd-5597-4209-aee0-5a5d145274a4.png)

## 2.1 Load Balancer 생성하기

EC2에서 Load Balancer를 생성합니다.

![image](https://user-images.githubusercontent.com/49144662/142973151-731f14f5-e5ab-4c71-b3a4-b84399ec4e32.png)

Application Load Balancer를 선택하여 HTTP 및 HTTPS 트래픽을 다루는 Load Balancer를 선택합니다.

![image](https://user-images.githubusercontent.com/49144662/142973189-22451a00-ce5f-4415-bb13-1b858ec446e1.png)

- 기본 구성
  - 이름: 로드밸런서 이름 지정
  - IP 주소 유형: ipv4로 설정합니다.
- 리스너: 리스너는 나중에 추가로 설정합니다.
- 가용 영역
  - VPC: 사용할 VPC 선택
  - 가용 영역: 이 영역에서 public subnet인 subnet을 선택합니다.

![image](https://user-images.githubusercontent.com/49144662/142973608-5690fb5e-1619-412b-944c-78215092e4d8.png)

2단계 스킵합니다.

![image](https://user-images.githubusercontent.com/49144662/142973667-95230d7b-0056-4256-b5b5-848e69da0b47.png)

보안 그룹에서 80번을 열고 0.0.0.0/0으로 설정합니다. 필요하면 22 ssh 포트도 열어두셔도 됩니다.

![image](https://user-images.githubusercontent.com/49144662/142973222-9c3b479c-365f-43e5-8c94-e329e444d105.png)

4단계 라우팅 구성

대상 그룹

- 이름: 그룹 이름 지정
- 대상 유형: 인스턴스로 선택
- 프로토콜: HTTP, HTTPS 중에 선택하는데 여기선 HTTP 선택
- 포트: 8080(스프링이라면 8080으로 설정)

상태 검사

- 프로토콜: HTTP로 설정
- 경로: / (여기서 Spring프로젝트/resource/static/index.html을 인지하여 연결확인합니다)
  - 또는 다른 컨트롤러 URL로 연결해도 괜찮습니다.

고급 상태 검사 설정

그대로 진행합니다.

## 2-b.1 ELB HTTPS 설정

### ELB 생성하기

**1단계: Load Balancer 구성**

![image](https://user-images.githubusercontent.com/49144662/142973235-3b3605ea-a341-461b-9641-a80de4e4a391.png)

HTTPS 프로토콜을 추가하고 Load Balancer 포트 443을 엽니다.

**2단계: 보안 설정 구성**

![image](https://user-images.githubusercontent.com/49144662/142973290-2c20bcce-e2b2-4aa4-a75e-fda72890eb46.png)

ACM 인증서를 선택합니다. ACM이 없다면 ACM을 생성하고 추가합니다.

[ACM 설정과 Route 53설정하기](https://www.sunny-son.space/AWS/Route53%20%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/)

**3단계: 보안 그룹 구성**

보안 그룹을 설정할 때 HTTPS를 추가하여 443포트도 열어두는 설정을 추가합니다.

**4단계: 라우팅 구성**

EC2 인스턴스와 ELB간의 통신은 8080으로 할 계획이기때문에 기존과 동일합니다.

**5단계: 대상 그룹 설정**

대상 그룹을 설정하지 않아도 됩니다. Auto Scaling AMI를 이용할 계획이기 때문에 설정할 필요가 없습니다.

### 리스너 설정하기

HTTP:80에서 리다이렉션합니다.

![image](https://user-images.githubusercontent.com/49144662/142973312-8a679738-96a5-45d1-9dbb-9132e1c45d4a.png)

- 규칙 보기/편집을 선택합니다.

![image](https://user-images.githubusercontent.com/49144662/142973331-cfcf15c7-8cfe-4c19-a3a4-d6086011093f.png)

- 규칙에서 THEN 부분만 설정하면 됩니다. 리디렉션 대상 HTTPS 443으로 설정하고 301 Status Code로 설정하고 저장하면 해당 ELB는 80으로 연결하더라도 자동으로 443으로 이동합니다.

## 2-b.2 Route 53 ELB 도메인 추가하기

![image](https://user-images.githubusercontent.com/49144662/142973341-f5eed494-57ef-4c86-bf13-ac7880bdd148.png)

- Route 53에 추가하기위해 DNS 이름을 복사합니다.

![image](https://user-images.githubusercontent.com/49144662/142973352-a7a61395-5dc8-4f79-8e45-64f89539bbff.png)

- Route 53에 해당 ELB의 DNS를 등록합니다. 레코드 유형은 CNAME으로 설정합니다.

다음은 AWS AutoScaling, Deploy, script를 이용하여 배포해보겠습니다!!
