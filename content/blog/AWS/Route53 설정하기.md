---
title: 'Route53 설정하기'
date: 2021-01-08
category: 'AWS'
draft: false
---

# 2. Route53 설정하기

도메인을 만드는 방법은 두가지로 나뉩니다. 도메인 구매처에서 사는법(예 - 가비아), AWS에서 도메인을 사는법이 있습니다. 여기선 AWS에서 구매하는법을 다룹니다.

## 2-1. 도메인 등록하기

![route53-01](https://user-images.githubusercontent.com/49144662/104004780-81171180-51e7-11eb-84e9-76469359afcd.png)

도메인 등록을 클릭하여 도메인을 등록합니다.

![route53-02](https://user-images.githubusercontent.com/49144662/104004856-9e4be000-51e7-11eb-879c-cbcb36f6e86d.png)

도메인 이름을 선택하고 구매를 합니다.

![route53-03](https://user-images.githubusercontent.com/49144662/104004868-a146d080-51e7-11eb-90a1-155811e9aa9e.png)

도메인 구입을 완료하면 구입하고나서 기다려야합니다. 기다려서 구입이 완료되면 AWS에서 알아서 도메인이 등록되니 도메인을 등록을 할 필요가 없습니다.

## 2-2. Route53과 S3 연결하기

Route53과 S3 연결은 어렵지 않습니다. 레코드 생성을 클릭합니다.

![route53-04](https://user-images.githubusercontent.com/49144662/104004874-a1df6700-51e7-11eb-8dae-8114c16a608f.png)

라우팅 정책에서 단순 라우팅으로 설정하고 다음을 클릭합니다.

![route53-05](https://user-images.githubusercontent.com/49144662/104004876-a277fd80-51e7-11eb-8aec-82177d02ec26.png)

레코드 구성을 합니다. 단순 레코드 정의를 클릭하여 S3와 연결합시다. S3에서 설정한 도메인이름으로 레코드이름을 정의합니다. 생성한 S3 도메인이 서브도메인(www나 blog(예 - cafe.daum.net 여기서 cafe가 서브 도메인입니다.) 

![route53-06](https://user-images.githubusercontent.com/49144662/104004878-a3109400-51e7-11eb-95c9-0e460123f186.png)

값/트래픽 라우팅 대상에서 S3 웹 사이트 엔드포인트로 설정하고 리전을 선택 → S3 버킷 선택

## 2-3. Redirect 설정하기

기존 도메인 http://example.com 은 설정했지만 www 서브 도메인은 생성하지 않았습니다. www 서브도메인으로 오는 경우에는 기존의 http://example.com 으로 Redirect 시켜봅시다.

### S3에서 서브도메인용 버킷 생성하기

기존의 정적 웹 사이트 호스팅용과 똑같습니다. 다른점은 버킷 정책을 생성할 필요가 없고, 속성에서도 요청 리다이렉션만 하면 됩니다.

![route53-07](https://user-images.githubusercontent.com/49144662/104004883-a3a92a80-51e7-11eb-985d-67e6603ba483.png)

프로토콜은 http 로 설정합니다.

### CNAME 사용하여 기존 도메인 연결하기

레코드 생성하고 레코드 이름에 서브도메인 www를 입력합니다.

값/트래픽 라우팅 대상에 Redirect 주소를 적습니다.

![route53-08](https://user-images.githubusercontent.com/49144662/104004884-a441c100-51e7-11eb-8ccb-23968e70dfa7.png)

정의완료하시면 기존의 도메인 + www.도메인 이 호스팅이 되었습니다.

만약 EC2와 같이 연동하신다면 다음 파트를 보시는걸 추천합니다.

## 2-4. EC2와 연결하기

EC2도 그리 어렵지 않습니다. 단지 필요한건 EC2에 탄력적 IP를 연결만 하고 Route53에서 설정을 하면됩니다.

서브도메인 명을 정하고(저는 backend라고 지었습니다.), 레코드 유형을 A로 설정합니다.

값/트래픽 라우팅 대상은 IP주소로 설정하고, 설정하고 싶은 EC2 인스턴스의 탄력적 IP를 연결하면 끝입니다.

![route53-09](https://user-images.githubusercontent.com/49144662/104004888-a441c100-51e7-11eb-8003-8c219e5b77ef.png)

거의 다왔군요:yum: 다음은 CloudFront를 사용하여 배포해보겠습니다.