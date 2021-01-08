---
title: '정적 페이지 웹 호스팅 - 3.CloudFront로 배포'
date: 2021-01-08 19:43:00
category: 'AWS'
draft: false
---
# 3. CloudFront 배포

## 3-1. ACM(Amazon Certificate Manager) 설정하기

### ACM 생성하기

CloudFront에서 버지니아 북부 이외에는 찾기 어렵기 때문에 버지니아 북부로 설정하시고 ACM을 생성하는걸 추천합니다.

인증서에서 인증서가 없는 환경이라고 생각하고 인증서를 요청합니다.

![01](https://user-images.githubusercontent.com/49144662/104009723-d60a5600-51ee-11eb-97b5-7348b8fc1a7e.png)

공인 인증서 요청을 선택합니다.

![02](https://user-images.githubusercontent.com/49144662/104009731-d86cb000-51ee-11eb-8dc0-0991527023a2.png)

도메인 이름을 추가합니다.

![03](https://user-images.githubusercontent.com/49144662/104009735-d99ddd00-51ee-11eb-8a2c-e81e7d5fbeae.png)

도메인이름을 *.example.com 으로 설정하면 www.example.com, blog.example.com 등등 여러 서브도메인에 SSL 추가가 가능합니다.

검증 방법 선택에서 선택합니다. Route53에서 이미 도메인을 등록했다면 DNS 검증을 선택합니다.

![04](https://user-images.githubusercontent.com/49144662/104009742-da367380-51ee-11eb-993a-99593edab590.png)

태그는 선택사항이라 바로 스킵하고 검토 및 요청에서 자신의 도메인 이름과 검증 방법을 확인합니다.

![05](https://user-images.githubusercontent.com/49144662/104009744-dacf0a00-51ee-11eb-9404-6535f523e18a.png)

여기까지 하고 확인 및 요청을 선택합니다.

몇 분 기다리시다가(최대 30분) 발급이 완료되었습니다! 

DNS 검증을 진행하지 않으면 검증 보류라고 상태표시가 되어있을 겁니다.

![06](https://user-images.githubusercontent.com/49144662/104009745-db67a080-51ee-11eb-82a7-cbec67fea441.png)

이럴 때는 아래와 같이 도메인 좌측의 화살표를 클릭하시면 Route 53에서 레코드 생성 버튼을 눌러 Route53 레코드 생성을 합니다.

![07](https://user-images.githubusercontent.com/49144662/104009746-db67a080-51ee-11eb-9223-9668a7425edb.png)

그림에서는 Route53이 비활성화 되어있지않지만 검증 보류상태일 땐 레코드 생성이 가능합니다.

해당 버튼을 선택하시면 Route53을 통해 자동으로 CNAME이 등록되며 DNS 검증이 진행됩니다.

![08](https://user-images.githubusercontent.com/49144662/104009748-dc003700-51ee-11eb-98da-4036e1205c06.png)

출처: [https://jojoldu.tistory.com/434](https://jojoldu.tistory.com/434)

자신의 Route53을 확인하면 CNAME이 생성된 것을 확인할 수 있습니다.

![09](https://user-images.githubusercontent.com/49144662/104009749-dc98cd80-51ee-11eb-9769-e83239438cf2.png)

최대 30분정도 기다리시면 Certificate Manager에서 발급 완료를 확인할 수 있습니다.

이로써 ACM 생성이 완료되었습니다. 생성한 ACM은 CloudFront나 EC2 로드밸런서 등에 사용이 가능합니다.

## 3-2. CloudFront 생성하기

Create Distribution을 눌러 생성합니다.

![10](https://user-images.githubusercontent.com/49144662/104009750-dc98cd80-51ee-11eb-846c-285c7dca889a.png)

웹을 선택 합니다.

![11](https://user-images.githubusercontent.com/49144662/104009751-dd316400-51ee-11eb-9001-cb52ccfce5ec.png)

여기서부터 항목이 많아서 세부적으로 설명하겠습니다.

![12](https://user-images.githubusercontent.com/49144662/104009753-dd316400-51ee-11eb-9c22-e951779436b2.png)

### Origin Settings

![13](https://user-images.githubusercontent.com/49144662/104009754-ddc9fa80-51ee-11eb-9d9e-e6d4e63c6b95.png)

- Origin Domain Name: S3 버킷엔드포인트를 그대로 입력하면 됩니다. 주의하실 점은 자동으로 생성해주는 S3 버킷엔드포인트가 아닌 리전까지 전부 표시해야 리다이렉트 할 때 작동이 올바르게 됩니다.

→ example.com.s3-website.us-east-1.amazonaws.com (O)
    example.com.s3.amazonaws.com (x)

- Origin Path: 비워둡니다.
- Origin ID: Origin Domain Name을 설정하면 자동으로 생성하는데 그대로 Default값으로 둡니다.
- Origin Connection Attempts: S3로 설정시 생성됩니다.
- Origin Connection Timeout: S3로 생성시 생성됩니다. 새로 생긴 이 두가지 항목은 그대로 Default값으로 둡니다.
- Origin Custom headers: 비워둡니다.

### Default Cache Behavior Settings

![14](https://user-images.githubusercontent.com/49144662/104009756-ddc9fa80-51ee-11eb-94ba-6e269c8ea711.png)

![15](https://user-images.githubusercontent.com/49144662/104009762-de629100-51ee-11eb-9862-16fab65e42d5.png)

- View Protocol Policy: HTTP, HTTPS 를 선택하는데 저는 HTTP요청이 오면  HTTPS로 리다이렉트하도록 설정했습니다.
- Allowed HTTP Methods: 그대로 GET, HEAD로 둡나다.
- Field-level Encryption Config: 디폴트 값으로 둡니다.
- Cache and origin request settings: 캐싱할 대상을 header로 구분하여 지정하는데 정책을 만들어 사용가능합니다. 저는 그대로 디폴트값으로 두었습니다.
- Cache Policy: 캐싱 정책. 디폴트값으로 두기.
- Origin Request Policy: Origin 헤더 추가가 가능합니다. 서버와 요청을 주고받을 때 CORS가 발생할 수 있으니 S3 Origin 설정을 합니다.

![16](https://user-images.githubusercontent.com/49144662/104009765-de629100-51ee-11eb-8502-3efd3c16a741.png)

- 나머지 대상은 그대로 Default값으로 두었습니다.

### Distribution Settings

![17](https://user-images.githubusercontent.com/49144662/104009766-defb2780-51ee-11eb-9946-96469ccca367.png)

![18](https://user-images.githubusercontent.com/49144662/104009769-df93be00-51ee-11eb-9fff-e6e0c7952917.png)

- Price Class: 그대로 디폴트값.
- Alternate Domain Names(CNAMEs): 여기서 도메인명을 그대로 적습니다. 서브도메인과 함께 적는게 아닌 지금은 도메인명만 그대로 적습니다. (예 - example.com)
- SSL Certificate: SSL 설정을 해야하는데 Custom SSL Certificate를 선택하여 ACM을 선택합시다. 만약 ACM의 리전이 US-East(Virginia)가 아니라면 뜨지 않으니 꼭 확인하길 바랍니다.
- Supported HTTP Versions: HTTP 버전선택하는데 디폴트값으로 HTTP/2까지 포함시킵니다.
- Default Root Object: S3에서 index.html로 설정했으니 여기도 index.html로 설정합니다.
- 나머지는 디폴트값으로 두었습니다.

여기까지 하면 기존 도메인 설정은 마쳤습니다. www 서브도메인 리다이렉션까지 설정하려면 CloudFront를 하나 더 생성하셔야합니다. 기본 원리는 다음과 같습니다.

![19](https://user-images.githubusercontent.com/49144662/104009771-df93be00-51ee-11eb-85cc-97d67499141e.png)

출처: [https://www.portzeroventures.com/blog/s3-website-complete-setup/](https://www.portzeroventures.com/blog/s3-website-complete-setup/)

### Redirect S3 추가하기

서브 도메인 www 도 똑같이 생성합니다. 

- Origin Domain Name S3 설정하기.
- Distribution Settings에서 CNAMEs에서 도메인 적기(www 포함하여적기).
- 단, Default Root Object에는 비워두기.

CloudFront를 배포할 때까지 시간이 걸리기 때문에 한번 설정할 때 잘 확인하면서 진행하는걸 추천합니다.

여기까지 하셨으면 현재 도메인, www를 사용한 서브도메인 까지 모두 CloudFront를 이용한 CDN 배포를 완료했습니다. 그다음은 Route53을 이용하여 도메인 연결을 마무리지으면 됩니다.

## 3-3. CloudFront와 Route53 연결하기

Route53에서 CloudFront와 연결하는것은 어렵지 않습니다.

Route53에서 S3와 연결된 레코드 이름을 선택하여 편집을 누릅니다.

![20](https://user-images.githubusercontent.com/49144662/104009772-e02c5480-51ee-11eb-981f-b78121b71493.png)

레코드 편집에서 값/트래픽 라우팅 대상만 편집하면 됩니다.

![21](https://user-images.githubusercontent.com/49144662/104009773-e02c5480-51ee-11eb-9099-0dea02a8c8fe.png)


CloudFront를 선택하시면, 버지니아 북부만 활성화가 됩니다. 그다음 연결하고자 하는 CloudFront 도메인이름을 잘 확인하셔서 선택하면 됩니다.

www 도메인도 똑같이 편집하여 S3에서 CloudFront연결을 하도록 설정합니다.

이제 CloudFront와 Route53 전부 연결을 마쳤습니다. 이후 몇 분 기다리시면 해당 주소로 정상적으로 되는걸 확인할 수 있습니다~~:smile: