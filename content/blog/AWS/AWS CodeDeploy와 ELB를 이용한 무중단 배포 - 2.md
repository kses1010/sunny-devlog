---
title: 'AWS CodeDeploy, ELB(L7)을 이용한 무중단 배포 - 2/2'
date: 2021-11-24
category: 'AWS'
draft: false
---

# 3. Auto Scaling 생성하기

![image](https://user-images.githubusercontent.com/49144662/142974693-34d72c6b-cd9f-40a9-a335-a3165bc71a24.png)

## 3.1 Auto Scaling - Launch Configurations


EC2 → Auto Scaling → Launch Configurations 에서 시작 구성 생성

![image](https://user-images.githubusercontent.com/49144662/142974705-a989e8bd-d6ad-446f-be08-4499c31f67ca.png)

- 시작 구성 이름: 이름 생성
- AMI: 이전에 만든 AMI를 선택합니다.
- 인스턴스 유형: 인스턴스 유형을 선택하는데 테스트용이라 t2.micro를 선택합니다.

![image](https://user-images.githubusercontent.com/49144662/142974715-8ea48c84-07ef-4ed1-b3e2-3891c87731b1.png)

- IAM 인스턴스 프로파일: IAM Role을 지정합니다.(CodeDeployRole)
- 고급 세부 정보에서 작성할 것은 사용자 데이터(user data)만 작성합니다.
- 사용자 데이터는 인스턴스 생성(AMI)시 실행할 명령어로 원본 인스턴스에서 미리 환경을 구축하면 편합니다.

```bash
#! /bin/bash

cd /home/ubuntu
./start.sh

# codeDeploy 실행
# 확인 결과 AMI에서 codeDeploy-agent가 설치되었다면 실행하지 않아도 자동으로 실행합니다.
sudo ./install auto
```

- 원본 인스턴스에서 필요한 환경
  - AWS CodeDeploy Agent
  - Java
  - Spring 프로젝트 구동 쉘 스크립트

![image](https://user-images.githubusercontent.com/49144662/142974735-64979c3f-f8f1-4249-be2f-eb090ce9c926.png)

- IP 주소 유형
  - Auto Scaling 대상 인스턴스를 private subnet으로 설정하기 때문에 어느 인스턴스에도 퍼블릭 IP 주소 할당하지 않습니다.
- 스토리지(볼륨): 원본 인스턴스의 크기로 지정합니다.

![image](https://user-images.githubusercontent.com/49144662/142974758-eddc5c10-c5ad-4e54-b439-cfb665c136eb.png)

- 보안 그룹: HTTP, 8080(loadbalancer의 소스와 연결), SSH(해당 인스턴스로 확인하고 싶을 경우)
- 키 페어: SSH 로그인을 할 경우 키 페어 선택. private subnet이므로 들어가려면 elastic ip를 설정하면 해당 인스턴스로 SSH 접근이 가능합니다.
- 또는 키 페어 없이 계속을 선택하여 ssh 인스턴스 접근을 막고 SSM으로 접근하는 방법도 있습니다.

## 3.2 Auto Scaling Group 설정하기

![image](https://user-images.githubusercontent.com/49144662/142974762-377ab1fd-f983-4940-9124-c10071134951.png)

그룹 생성하기

![image](https://user-images.githubusercontent.com/49144662/142974776-84a5b417-8146-4c18-b581-caa0b4c3d375.png)

![image](https://user-images.githubusercontent.com/49144662/142974789-396fb9aa-0015-4be5-999a-fb88036fb41e.png)

- 이름: 그룹 이름 설정
- 시작 구성: 시작 구성으로 전환하여 이전에 Launch Configurations에서 만든 설정을 가져옵니다.

![image](https://user-images.githubusercontent.com/49144662/142974804-c3c7726d-3131-4639-a8f4-a455d6fe8cad.png)

- 서브넷: private subnet(NAT gateway로 연결)

![image](https://user-images.githubusercontent.com/49144662/142974815-d22bc4fd-e69d-4a96-83e1-aef4013f6c7c.png)

- 로드 밸런싱: 기존 로드 밸런서에 연결
- 기존 로드 밸런서에 연결: 이전에 만든 로드밸런서를 사용합니다.

![image](https://user-images.githubusercontent.com/49144662/142974824-52ecd926-b700-40bd-8f7b-f6502f02571c.png)

- 그룹 크기: 새로운 인스턴스를 생성합니다. 로드밸런싱이 잘됐는지 테스트하기 위해 최소, 최대 모두 2로 설정합니다.
- 조정 정책: 없음

![image](https://user-images.githubusercontent.com/49144662/142974839-1e29e2d1-1fef-413e-8bc5-0538e2247e82.png)

- 태그 추가: 새 인스턴스에 어떤 이름을 붙일지 지정해줍니다.

![image](https://user-images.githubusercontent.com/49144662/142974846-7b504b16-1306-4938-9a5b-d79204168eec.png)

해당 그룹에서 새로운 인스턴스를 확인할 수 있습니다.

------

# 4. AWS CodeDeploy 설정하기

![image](https://user-images.githubusercontent.com/49144662/142975075-e1de5f52-c7ad-41e3-a5d5-812633876b70.png)

AWS CodeDeploy에서 생성합니다.

![image](https://user-images.githubusercontent.com/49144662/142975087-49c83322-3a0f-4cac-b797-6b52aeb7d7e3.png)

- EC2/온프레미스를 선택합니다.

![image](https://user-images.githubusercontent.com/49144662/142975101-cd3b511e-d7b0-4671-9759-605ab6d36161.png)

- 배포 그룹을 생성합니다.
- 서비스 역할: CodeDeploy Role을 추가합니다.

![image](https://user-images.githubusercontent.com/49144662/142975127-55e2d5b7-b10e-4c82-8e5c-bfbb43a834d0.png)

- 애플리케이션 배포 방법: 블루/그린 선택
- 환경 구성: 이전 생성한 Auto Scaling 그룹을 선택합니다.

![image](https://user-images.githubusercontent.com/49144662/142975142-4d740273-5161-4cb7-befd-bcb76ef5320d.png)

- 배포 설정
  - 트래픽 재 라우팅: 새로운 인스턴스 생성하자 마자 바로 라우팅 시작합니다.
  - 배포 그룹의 원본 인스턴스 종료: 대기시간을 지정하는 것으로 기본값은 1시간입니다. 테스트 환경을 위해 0일 0시간 0분으로 설정합니다.
  - 배포 구성: CodeDeployDefault.AllAtOnce를 선택.
- 로드 밸런서: 이전에 만든 로드밸런서(Application Load Balancer(HTTP, HTTPS))를 선택합니다.

여기까지 왔으면 AWS Console로 생성은 모두 마쳤습니다. 마지막으로 해당 프로젝트 루트에서 쉘스크립트, appspec.yml을 설정합니다.

# 5. 프로젝트 루트 설정

![image](https://user-images.githubusercontent.com/49144662/142975156-940c88ff-8aa7-456b-9cc2-e87598f9b08a.png)

## 5.1 Github Actions 설정

```yaml
- name: Code Deploy
  run: aws deploy create-deployment --application-name cicdtest --deployment-config-name CodeDeployDefault.OneAtATime --deployment-group-name cicd-bluegreen --s3-location bucket=cicdtestsunny,bundleType=zip,key=build.zip
```

- `--application-name` : AWS CodeDeploy 애플리케이션 이름
- `--deployment-config-name` : 배포 구성(CodeDeploy의 배포구성)
- `--deployment-group-name` : 배포 그룹 이름
- `--s3-location` : s3 버킷 이름, 빌드 타입, 빌드 파일이름 지정

## 5.2 appspec.yml 설정

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /home/ubuntu/myapp # 파일 배포 경로
permissions:
  - object: /home/ubuntu/
    owner: ubuntu
    group: ubuntu
    mode: 755

hooks:
  BeforeInstall:
    - location: beforeInstall.sh # 배포 이전
      timeout: 60
      runas: root
  AfterInstall:
    - location: deploy.sh
      timeout: 60
      runas: root
```

- hooks

![image](https://user-images.githubusercontent.com/49144662/142975173-662e2836-b129-456b-848b-4d6041e38f9a.png)

AWS CodeDeploy는 이전 파일이 존재일 경우 에러가 발생합니다.

```
The deployment failed because a specified file already exists at this location
```

CodeDeploy는 파일 덮어쓰기가 불가능하기 때문에 다음 레퍼런스처럼 삭제하고 다시 배포하는 형식을 취하면 됩니다.

> 출처: [Add Support for "Overwrite" instruction in appspec.yml "Files" section · Issue #14 · aws/aws-codedeploy-agent](https://github.com/aws/aws-codedeploy-agent/issues/14)

### [beforeInstall.sh](http://beforeInstall.sh)

```bash
#!/usr/bin/env bash

REPOSITORY=/home/ubuntu/

if [ -d $REPOSITORY/myapp ]; then
    rm -rf $REPOSITORY/myapp
fi
mkdir -vp $REPOSITORY/myapp
```

- 이전 파일이 있다면 삭제하고 다시 생성

### [deploy.sh](http://deploy.sh)

```bash
#!/usr/bin/env bash

REPOSITORY=/home/ubuntu/myapp
APP_NAME=demo

echo "> pid 확인"
CURRENT_PID=$(pgrep -fl $APP_NAME | grep java | awk '{print $1}')

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
  echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $CURRENT_PID"
  kill -15 $CURRENT_PID
  sleep 5
fi

echo "> 새 애플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> Jar name: JAR_NAME"
echo "> $JAR_NAME에 실행 권한 추가"
chmod +x $JAR_NAME

echo "> $JAR_NAME 배포"
nohup java -jar $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

배포스크립트는 해당 프로젝트마다 다르니 꼭 쉘스크립트는 프로젝트 환경 확인하시고 작성해주세요.

------

여기까지 작성하느라 고생하셨습니다. 이제 테스트를 해보겠습니다.

# 6. Blue/Green 테스트

우선 간단한 Spring 컨트롤러를 만듭니다.
```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.net.InetAddress;
import java.net.UnknownHostException;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String getUser() {

        String hostName, ipaddress;

        try {
            hostName = InetAddress.getLocalHost().getHostName();
            ipaddress = InetAddress.getLocalHost().getHostAddress();
        } catch (UnknownHostException e) {
            hostName = new StringBuilder("Error : ").append(e.getLocalizedMessage()).toString();
            ipaddress = "";
        }

        return "hostName : " + hostName + " ipaddress : " + ipaddress + " pland가 조아";
    }

    @GetMapping("/hi")
    public String hi() {
        String hostName;
        String ipaddress;

        try {
            hostName = InetAddress.getLocalHost().getHostName();
            ipaddress = InetAddress.getLocalHost().getHostAddress();
        } catch (UnknownHostException e) {
            hostName = new StringBuilder("Error : ").append(e.getLocalizedMessage()).toString();
            ipaddress = "";
        }

        return "Hi" + hostName + " :: " + ipaddress;
    }
}
```

hostname, piaddress를 생성하여 auto scaling과 load balancer가 제대로 작동하는지 확인할 수 있는 컨트롤러 입니다.

커밋하고 실행!

![image](https://user-images.githubusercontent.com/49144662/142975195-95d3adca-3115-474a-afcb-20c34f31808b.png)

다음과 같이 인스턴스 생성 삭제를 확인할 수 있습니다.
![image](https://user-images.githubusercontent.com/49144662/142975211-a40cbf80-a83f-4e83-99f7-6d916c3abe46.png)

ELB Round Robin 방식으로 새로고침할 때마다 ip가 달라지는 모습을 확인할 수 있습니다.

### HTTPS에 대하여

기존의 Classic Load Balancer는 스프링에서 처리할 때 HTTP로 요청을 받아 처리하는 바람에 OAuth2 로그인 적용시 HTTPS로 요청보냈다가 HTTP로 받을 수도 있었습니다. 그래서 스프링부트의 내부톰캣 설정으로 다음과 같은 처리가 필요합니다.

```
server.tomcat.remoteip.protocol-header=x-forwarded-proto
```

> 출처: [SpringSecurity via ELB , Login redirect to http](https://circlee7.medium.com/springsecurity-via-elb-login-redirect-to-http-54f0805cb408)

> 출처: [HTTPS/SSL/Spring Security doesn't work in both a load balancer and non-load balancer environment · Issue #424 · BroadleafCommerce/BroadleafCommerce](https://github.com/BroadleafCommerce/BroadleafCommerce/issues/424)

그러나 현재 사용하고 있는 Application Load Balancer는 전부 HTTPS로 처리하기 때문에 스프링 톰캣 설정을 추가할 필요가 없습니다.

![image](https://user-images.githubusercontent.com/49144662/142975229-dfc1a499-1b4d-45e5-a10b-25d178411b62.png)

여기까지 따라오시느라 고생많으셨습니다!! 

ELB로 무중단 배포를 성공적으로 이뤄지길 바랍니다!!😆

### 배포 후기

Nginx와는 다르게 AWS 인프라 하나로 다 처리할 수 있어 관리할 때는 편합니다.

다만, 시간이 Nginx 배포보다 훨씬 느리고 비용도 더 많이 들어 스타트업이나 작은 규모의 회사에서는 사용할 가치가 있는지 고려를 해야할 듯 합니다.

특히 배포 테스트할 때 너무 오래걸려 힘들었어요😭

## TroubleShooting

### AWS CodeBuild

- DOWNLOAD_SOURCE 실패할 때

  - `CLIENT_ERROR: Get "[<https://github.com/pland-sunny/cicdtest/info/refs?service=git-upload-pack>](<https://github.com/pland-sunny/cicdtest/info/refs?service=git-upload-pack>)": dial tcp 52.78.231.108:443: i/o timeout for primary source`

  → NAT Gateway 필요합니다.

### AWS CodeDeploy

- 만약 CodeDeploy에서 실패하여 이벤트 로그를 확인했는데 아무내용이 없다?

→ IAM을 권한 받기전에 AWS CodeDeploy를 실행했기 때문입니다.

→ `sudo service codedeploy-agent restart` 를 시도해볼것!

```
codedeploy The specified key does not exist
```

해당 파일이 없다는 뜻으로 S3에서 파일이름, 확장자 확인해야합니다.

```
The deployment failed because no instances were found in your blue fleet.
```

blue 인스턴스가 생성하지 못함 → 확인 결과 2b의 서브넷은 t2.micro 생성하도록 설정했기 때문에 auto-scaling에서 생성하지 못하여 에러 발생합니다.

- 인스턴스 Security-group은 22, 8080 열기(해당 소스는 Load Balancer로 향하여)
- Load Balancer에서 해당 인스턴스 연결할 때 80포트로 연결할 것!!

```
The deployment failed because a specified file already exists at this location
```

- BeforeInstall을 설정하여 기존배포파일을 삭제 또는 옮긴 후 실행하도록 설정할 것!!
