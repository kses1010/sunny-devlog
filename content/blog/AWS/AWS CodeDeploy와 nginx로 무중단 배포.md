---
title: 'AWS CodeDeploy + Nginx로 무중단 배포'
date: 2021-04-04
category: 'AWS'
draft: false
---

# AWS CodeDeploy + Nginx 무중단 배포 아키텍처

![01](https://user-images.githubusercontent.com/49144662/113500039-fe87bf00-9555-11eb-875a-5510621b3b77.png)

# 0. 도입 배경

AWS Blue/Green 무중단 배포방식은 ELB와 EC2 인스턴스 하나 더 필요합니다. 비용이 많이 필요하다보니 또다른 방법인 Nginx를 이용한 무중단 배포 방식을 사용하기로 했습니다.

Nginx를 이용한 무중단 배포는 AWS EC2 인스턴스가 하나 더 필요하지 않고, 꼭 AWS와 같은 클라우드 인프라가 필요하지 않고 사용할 수 있는 방법 중 하나입니다.

구조는 간단하게 EC2 인스턴스 1대, Nginx 1대, 스프링 부트 jar를 2대 사용하면 됩니다.

## 0.1 AWS CodeBuild + Github Actions

[Github Actions를 이용한 AWS CodeBuild](https://www.sunny-son.space/AWS/AWS%20CodeBuild/)

# 1. Nginx 설정하기

## 1.1 Nginx 설치

EC2에 접속하여 Nginx를 설치합니다.

```bash
# nginx 설치
sudo apt-get install nginx
# nginx 실행
sudo service nginx start
# nginx 확인
ps -ef | grep nginx
```

## 1.2 Nginx 설정

```bash
# nginx 설정폴더로 이동
cd /etc/nginx/sites-available
# nginx 설정 수정하기
sudo vi default
```

![02](https://user-images.githubusercontent.com/49144662/113500041-00ea1900-9556-11eb-818b-8546cce99d4e.png)

```bash
proxy_pass http://localhost:8080;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Host $http_host;
```

- proxy_pass: `http://localhost:8080`
- proxy_set_header XXX : 실제 요청 데이터를 header의 각 항목에 할당
  - ex) `proxy_set_header X-Real-IP $remote_addr`: Request Header의 X-Real-IP에 요청자의 IP를 저장합니다.

설정을 마쳤으면 다음 명령어를 내려 Nginx를 재시작합니다.

```bash
sudo service nginx restart
```

# 2. Spring Boot Profile 설정하기

![03](https://user-images.githubusercontent.com/49144662/113500042-0182af80-9556-11eb-8950-a4b365899a0d.png)

- yaml로 설정하는 방법도 있지만 여기선 properties방식으로 진행하겠습니다.

실제 스프링부트 프로젝트를 실행할 때

```bash
nohup java -jar -Dspring.profiles.active={프로퍼티이름}
```

사용하여 프로퍼티를 선택하여 실행이 가능합니다.

예시

```bash
nohup java -jar -Dspring.profiles.active=dev
```

각 properties마다 port번호를 다르게 부여합니다.

```bash
# dev
server.port=8081

# dev2
server.port=8082
```

실행하고 있는 프로젝트의 Profile을 확인할 수 있는 API를 만듭니다.

```java
@RestController
public class HelloController {

    private final Environment env;

    public HelloController(Environment env) {
        this.env = env;
    }

		@GetMapping("/profile")
    public String getProfile () {
        return Arrays.stream(env.getActiveProfiles())
            .findFirst()
            .orElse("");
    }
```

![04](https://user-images.githubusercontent.com/49144662/113500043-0182af80-9556-11eb-8e65-b05690b1bcaf.png)

![05](https://user-images.githubusercontent.com/49144662/113500045-021b4600-9556-11eb-9236-4f753af10b7c.png)

- dev로 바꿔 실행해보면 port가 8080이 아닌 8081인 것을 확인할 수 있습니다.

# 3. 스크립트 작성하기

## 3.1 배포 스크립트 작성

```bash
#!/usr/bin/env bash

BASE_PATH=/home/ubuntu/
BUILD_PATH=$(ls $BASE_PATH/myapp/*.jar)
JAR_NAME=$(basename $BUILD_PATH)
echo "> build 파일명: $JAR_NAME"

echo "> build 파일 복사"
DEPLOY_PATH=/home/ubuntu/temp/
cp $BUILD_PATH $DEPLOY_PATH

echo "> 현재 구동중인 Set 확인"
CURRENT_PROFILE=$(curl -s http://localhost/profile)
echo "> $CURRENT_PROFILE"

# 쉬고 있는 dev 찾기: dev이 사용중이면 dev2가 쉬고 있고, 반대면 dev이 쉬고 있음
if [ $CURRENT_PROFILE == dev ]
then
  IDLE_PROFILE=dev2
  IDLE_PORT=8082
elif [ $CURRENT_PROFILE == dev ]
then
  IDLE_PROFILE=dev
  IDLE_PORT=8081
else
  echo "> 일치하는 Profile이 없습니다. Profile: $CURRENT_PROFILE"
  echo "> dev 할당합니다. IDLE_PROFILE: dev"
  IDLE_PROFILE=dev
  IDLE_PORT=8081
fi

echo "> application.jar 교체"
IDLE_APPLICATION=$IDLE_PROFILE-demo.jar
IDLE_APPLICATION_PATH=$DEPLOY_PATH$IDLE_APPLICATION

# 미연결된 Jar로 신규 Jar 심볼릭 링크 (ln)
ln -Tfs $DEPLOY_PATH$JAR_NAME $IDLE_APPLICATION_PATH

echo "> $IDLE_PROFILE 에서 구동중인 애플리케이션 pid 확인"
IDLE_PID=$(pgrep -f $IDLE_APPLICATION)

if [ -z $IDLE_PID ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 $IDLE_PID
  sleep 5
fi

echo "> $IDLE_PROFILE 배포"
nohup java -jar -Dspring.profiles.active=$IDLE_PROFILE $IDLE_APPLICATION_PATH > $DEPLOY_PATH/nohup.out 2>&1 &

# Nginx Port 스위칭을 위한 스크립트
echo "> 스위칭"
sleep 10
/home/ubuntu/myapp/switch.sh
```

## 3.2 Nginx port 스위칭 스크립트

```bash
#!/usr/bin/env bash

echo "> 현재 구동중인  Port 확인"
CURRENT_PROFILE=$(curl -s http://localhost/profile)

if [ $CURRENT_PROFILE == dev ]
then
  IDLE_PORT=8082
elif [ $CURRENT_PROFILE == dev2 ]
then
  IDLE_PORT=8081
else
  echo "> 일치하는 Profile이 없습니다. Profile: $CURRENT_PROFILE"
  echo "> 8081을 할당합니다."
  IDLE_PORT=8081
fi

echo "> 전환할 Port: $IDLE_PORT"
echo "> Port 전환"
echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" |sudo tee /etc/nginx/conf.d/service-url.inc

PROXY_PORT=$(curl -s http://localhost/profile)
echo "> Nginx Current Proxy Port: $PROXY_PORT"

echo "> Nginx Reload"
sudo service nginx reload # reload는 설정만 재적용하기 때문에 바로 적용이 가능합니다.
```

- 스위치 스크립트는 Nginx에서 port를 바꾸는 스크립트입니다.
- 쉘 스크립트는 이동욱님 블로그(jojoldu)를 참고했습니다.

[7) 스프링부트로 웹 서비스 출시하기 - 7. Nginx를 활용한 무중단 배포 구축하기](https://jojoldu.tistory.com/267)

# 4. Nginx 동적 프록시 설정하기

```bash
cd /etc/nginx
```

![06](https://user-images.githubusercontent.com/49144662/113500047-02b3dc80-9556-11eb-8955-6dd66cb78fb2.png)

- `conf.d` 폴더에서 `service-url.inc` 를 생성합니다.

```bash
sudo vi service-url.inc

## service-url.inc
set $service_url http://127.0.0.1:8081;
```

설정을 추가합니다. (sites-available/default)

![07](https://user-images.githubusercontent.com/49144662/113500048-02b3dc80-9556-11eb-91f9-2de223ccf3bd.png)

```bash
include /etc/nginx/conf.d/service-url.inc;
proxy_pass $service_url;
```

저장하셨으면 다시 Nginx를 재시작합니다.

```bash
sudo service nginx restart
```

테스트를 위해 `curl` 을 수행합니다.

```bash
curl -s localhost/profile
```

![08](https://user-images.githubusercontent.com/49144662/113500049-034c7300-9556-11eb-99b1-48e2369187ab.png)

Nginx가 port:8081로 Proxy가 가는 것을 확인할 수 있습니다.

# 5. AWS CodeDeploy 설정하기

## 5.1 IAM 설정하기

![09](https://user-images.githubusercontent.com/49144662/113500051-034c7300-9556-11eb-99d9-fbfb9a30d31e.png)

![10](https://user-images.githubusercontent.com/49144662/113500053-03e50980-9556-11eb-8767-61a6dd31d812.png)

- CodeDeploy의 IAM 역할 설정: CodeDeployFullAccess 추가

## 5.2 IAM 역할 EC2 설정하기

![11](https://user-images.githubusercontent.com/49144662/113500086-3131b780-9556-11eb-9114-2c918462b404.png)

![12](https://user-images.githubusercontent.com/49144662/113500057-047da000-9556-11eb-80a1-959ccc0445da.png)

- 지정한 IAM CodeDeploy 역할을 붙입니다.

## 5.3 AWS CodeDeploy 추가하기

이전 포스트참고 하셔도 됩니다!

[Github Actions, AWS Deploy로 CI/CD해보자!](https://www.sunny-son.space/AWS/Github%20Action%20CICD/)

CodeDeploy 애플리케이션 생성

![13](https://user-images.githubusercontent.com/49144662/113500058-047da000-9556-11eb-9ca4-7456715b2de1.png)

![14](https://user-images.githubusercontent.com/49144662/113500059-05163680-9556-11eb-86a1-f258f8ba7afa.png)

- 컴퓨터 유형: EC2/온프레미스
- 애플리케이션 이름: 꼭 기억해야합니다!! Github Action workflow 사항에 기입해야하기 떄문이죠.

![15](https://user-images.githubusercontent.com/49144662/113500060-05163680-9556-11eb-98e0-aeb19400a2be.png)

- 배포 그룹 이름: 여기도 기억해야할 대상. workflow 기입대상입니다.
- 서비스 역할 설정: CodeDeploy역할 선택

![16](https://user-images.githubusercontent.com/49144662/113500061-05aecd00-9556-11eb-820b-870683d2b0ce.png)

- 배포방법 선택: 현재위치
- 환경구성
  - Amazon EC2 인스턴스
  - 키-값: EC2인스턴스 이름보고 선택

![17](https://user-images.githubusercontent.com/49144662/113500063-05aecd00-9556-11eb-9953-c731379af9a5.png)

- 배포 설정: CodeDeployDefault.AllAtOnce 선택
- 로드 밸런싱 비활성화 (설정하지 않았기 때문에 선택하지 않겠습니다)

# 6. Github Actions workflow 설정하기

```yaml
name: CD with Gradle

on:
  push:
    branches: [master]

jobs:
  buildAndTest:
    name: Github actions CD
    runs-on: ubuntu-18.04
    defaults:
      run:
        shell: bash

    steps:
      - name: 체크아웃 Github-Action
        uses: actions/checkout@v2

      - name: AWS 설정
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2

      - name: CodeBuild 실행
        uses: aws-actions/aws-codebuild-run-build@v1.0.3
        with:
          project-name: cicdtest

      - name: Code Deploy
        run: aws deploy create-deployment --application-name cicdtest --deployment-config-name CodeDeployDefault.OneAtATime --deployment-group-name nginx-bluegreen --s3-location bucket=cicdtestsunny,bundleType=zip,key=build.zip
```

```yaml
--deployment-config-name: 배포설정
--deployment-group-name: 배포그룹 이름
--s3-location : bucket=버킷이름, bundleType=파일 타입, key=파일 이름

key는 build.zip가 아닌 server/build.zip로 설정해야합니다.
```

# 7. CodeDeploy 대상 설정하기

- appspec.yml

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /home/ubuntu/myapp

permissions:
  - object: /home/ubuntu/
    owner: ubuntu
    group: ubuntu
    mode: 755

hooks:
  BeforeInstall:
    - location: beforeInstall.sh
      timeout: 60
      runas: ubuntu
  AfterInstall:
    - location: deploy.sh
      timeout: 60
      runas: ubuntu
```

overwrite에러가 발생할 가능성이 있기 때문에 `BeforeInstall` 단계를 거칩니다.

- beforeInstall.sh

```bash
#!/usr/bin/env bash

REPOSITORY=/home/ubuntu/

if [ -d $REPOSITORY/myapp ]; then
    rm -rf $REPOSITORY/myapp
fi
mkdir -vp $REPOSITORY/myapp
```

여기까지 설정하셨으면 CodeDeploy + Github Actions CI/CD 파이프라인은 모두 갖췄습니다. 이제 테스트를 실행합니다.

# 테스트!

## Github Actions 시작

![18](https://user-images.githubusercontent.com/49144662/113500064-06476380-9556-11eb-951a-ab100dfa1627.png)

![19](https://user-images.githubusercontent.com/49144662/113500065-06476380-9556-11eb-8fcb-00d741746cc0.png)

## CodeDeploy

![20](https://user-images.githubusercontent.com/49144662/113500066-06dffa00-9556-11eb-8a5d-1051d9412691.png)

배포 성공했습니다!

다음에는 로드밸런서를 이용하여 Blue/Green 배포를 해보겠습니다! 😆

# Troubleshooting

## Nginx

- 환경변수 세팅도 중요하다.

```bash
# dev 인스턴스
export DEFAULT_PROFILE=dev
echo $DEFAULT_PROFILE
dev

# live 인스턴스
export DEFAULT_PROFILE=prod
echo $DEFAULT_PR
```

[[Linux] 쉘 환경 변수](https://neul-carpediem.tistory.com/78)

- service-url.inc 파일 설정은 localhost로 설정하지 말고 127.0.0.1 로 설정할 것

쉘스크립트 실행 전에 디렉토리를 미리 만들어야합니다.

- deploy (CodeDeploy 대상 폴더 - appspec.yml, 쉘스크립트 등등)
- deployanother (복사 대상)
- logs (로그파일)
