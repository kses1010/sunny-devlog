---
title: 'Github Actions와 CodeBuild로 CI Build 환경 설정하기'
date: 2021-03-28
category: 'AWS'
draft: false
---

# 0. 도입배경

Jenkins의 비용과 성능 문제로 인해 다른 CI/CD툴을 찾기로 결정했습니다.

Trigger는 Github Actions를 사용하고 Build는 AWS CodeBuild, Deploy는 AWS CodeDeploy를 선택했습니다. AWS 설계는 다음과 같습니다.

![image](https://user-images.githubusercontent.com/49144662/112752923-58e1c680-9010-11eb-9b25-008f0002fadc.png)

# 1. AWS VPC 설정

![image](https://user-images.githubusercontent.com/49144662/112752932-6bf49680-9010-11eb-947d-dc68dcc741cd.png)

AWS CodeBuild는 NAT Gateway를 사용하여 Private Subnet에서 작동합니다. CodeBuild를 만들기 전에 VPC에서 Subnet 설정을 우선으로 합니다.

## 1.1 Subnet 생성

![image](https://user-images.githubusercontent.com/49144662/112752939-7747c200-9010-11eb-8a89-4bc59b4b1f4b.png)

![image](https://user-images.githubusercontent.com/49144662/112752960-8d558280-9010-11eb-8170-9fe64dc8010f.png)

VPC를 지정하고 서브넷 세팅합니다.

![image](https://user-images.githubusercontent.com/49144662/112752971-9e9e8f00-9010-11eb-81e4-647dc6281b0e.png)

- 서브넷 이름: 구분할 수 있는 이름 지정.
- 가용 영역: 가용 영역을 지정합니다. 여기서는 AWS가 2b, 2d지역에선 못만들게 하니 2a, 2c로 지정하겠습니다.
- IPv4 CIDR: 서브넷 블록이 서로 겹치지 않게 설정.

저는 2개의 서브넷을 생성했습니다.

## 1.2 NAT 게이트웨이 생성

![image](https://user-images.githubusercontent.com/49144662/112752978-ad854180-9010-11eb-8c0b-eecd73d23eed.png)

NAT 게이트웨이를 생성합니다.

![image](https://user-images.githubusercontent.com/49144662/112752987-c130a800-9010-11eb-9d84-868a2c3ec075.png)

- 이름: 구분할 수있는 이름을 짓습니다.
- 서브넷: NAT 게이트웨이를 생성할 **퍼블릭** 서브넷을 선택합니다.
- 탄력적 IP 할당: elastic IP를 설정합니다.

## 1.3 라우팅 테이블 설정하기

![image](https://user-images.githubusercontent.com/49144662/112753002-cee62d80-9010-11eb-9289-4898b91eaa6c.png)

라우팅 테이블을 생성하고 태그 추가해서 Name을 지정합니다.

![image](https://user-images.githubusercontent.com/49144662/112753014-e1f8fd80-9010-11eb-9dd9-fe17b607400b.png)

라우팅 편집을 합니다.

![image](https://user-images.githubusercontent.com/49144662/112753028-f4733700-9010-11eb-94bd-17508fda3223.png)

**라우팅 편집**

![image](https://user-images.githubusercontent.com/49144662/112753071-1967aa00-9011-11eb-8394-152fd463c5ef.png)

- 대상: 0.0.0.0/0 으로 설정
- NAT-Gateway로 설정합니다.

→ 서브넷 연결 편집

![image](https://user-images.githubusercontent.com/49144662/112753249-c6422700-9011-11eb-8a3b-6e39f9076f7a.png)

NAT Gateway에서 private 서브넷 대상만 추가합니다.

![image](https://user-images.githubusercontent.com/49144662/112753103-369c7880-9011-11eb-8c33-bf63142667c6.png)

# 2. AWS CodeBuild 설정하기

![image](https://user-images.githubusercontent.com/49144662/112753130-46b45800-9011-11eb-8984-caa0639951cc.png)

## 2.1 AWS IAM 역할 생성하기

![image](https://user-images.githubusercontent.com/49144662/112753146-5633a100-9011-11eb-8803-5089471968b8.png)

권한 정책에서 codebuild를 연결합니다.

![image](https://user-images.githubusercontent.com/49144662/112753157-66e41700-9011-11eb-991e-1477e67d1335.png)

역할 이름을 Codebuild로 명확히 합니다.

![image](https://user-images.githubusercontent.com/49144662/112753190-824f2200-9011-11eb-952f-8826a77e008b.png)

## 2.2 AWS CodeBuild 만들기

![image](https://user-images.githubusercontent.com/49144662/112753195-8b3ff380-9011-11eb-8b48-a54bd724578f.png)

프로젝트 이름을 지정합니다.

![image](https://user-images.githubusercontent.com/49144662/112753203-9561f200-9011-11eb-8a89-447f340c02d5.png)

소스 공급자에서 Github를 연결합니다.

![image](https://user-images.githubusercontent.com/49144662/112753315-15885780-9012-11eb-9216-594671ed1f0e.png)

빌드 환경을 설정합니다.

![image](https://user-images.githubusercontent.com/49144662/112753338-2933be00-9012-11eb-9cb7-8e48f2d950d5.png)

- 환경 이미지: AWS CodeBuild에서 관리 선택
- 운영체제: 아마존 리눅스, Ubuntu 중에서 선택
- 런타임, 이미지, 이미지 버전[https://docs.aws.amazon.com/ko_kr/codebuild/latest/userguide/build-env-ref-available.html](https://docs.aws.amazon.com/ko_kr/codebuild/latest/userguide/build-env-ref-available.html)
  해당 링크에서 확인합니다.

![image](https://user-images.githubusercontent.com/49144662/112753358-3f417e80-9012-11eb-8818-ab0a848df729.png)

- 서비스 역할 → 기존 서비스 역할 → 역할 ARN에서 이전의 IAM 설정을 추가합니다.

---

Additional configuration 설정

![image](https://user-images.githubusercontent.com/49144662/112753369-4f595e00-9012-11eb-956e-aa9addd7a326.png)

- Timeout: 타임아웃 설정하기
- VPC
- Subnets: 이전에 설정한 NAT 서브넷을 추가합니다.
- Security groups: NAT 서브넷에 설정한 Security group으로 설정합니다.
  - 전부 설정이 완료되었다면 Validate VPC Settings를 눌러 확인합시다.

![image](https://user-images.githubusercontent.com/49144662/112753376-584a2f80-9012-11eb-9c12-ea2cb1d5ab27.png)

**초록색으로 확인이 되었다면?**

- compute: 빌드 환경을 결정합니다.
- Environment variables: 환경변수 설정
- File systems

![image](https://user-images.githubusercontent.com/49144662/112753393-6e57f000-9012-11eb-804d-58eb9cdacfd2.png)

- Buildspec: buildspec 파일 사용합니다. → Github 루트에 buildspec.yml을 생성하면 됩니다.
- 배치 구성: 배치할 때 사용하면 됩니다.

![image](https://user-images.githubusercontent.com/49144662/112753403-76179480-9012-11eb-9f22-0e278b1d10e7.png)

- 아티팩트

  - 유형: S3로 빌드파일을 보내기로 설정합니다.
  - 버킷 이름: S3 버킷이름
  - 이름: 빌드된 파일이름 (ex: build.zip)
  - 경로: 경로는 선택하지 않았습니다. 여러버전의 빌드파일을 선택하려면 경로를 선택해도 됩니다.
  - 아티팩트 패키징: ZIP를 선택합니다.
  - 추가 구성으로 캐시를 사용하려면 다음과 같이 설정합니다.

![image](https://user-images.githubusercontent.com/49144662/112753411-7e6fcf80-9012-11eb-8f0f-fe6487cfe790.png)

- 캐시 유형: Amazon S3
- 캐시 경로 접두사는 cache로 설정합니다.

- 로그: 로그는 빌드 출력 로그를 S3에 저장할 것인지 선택하고 생성합니다.

## 2.3 buildspec.yml 설정하기

![image](https://user-images.githubusercontent.com/49144662/112753416-8760a100-9012-11eb-98e5-d22be5ef7537.png)

다음과 같이 루트에서 buildspec.yml을 생성합니다.

```yaml
version: 0.2

phases:
  build:
    commands:
      - chmod +x ./gradlew
      - ./gradlew build
  post_build:
    commands:
      - echo $(basename ./build/libs/*.jar)
      - pwd

artifacts:
  files:
    - build/libs/*.jar
    - appspec.yml
    - beforeInstall.sh
    - deploy.sh
  discard-paths: yes
cache:
  paths:
    - '/root/.gradle/caches/**/*'
```

- commands: 빌드 명령어입니다. gradle로 빌드하기 위한 명령어를 사용했습니다.
- post_build: 빌드 이후 명령어로 해당 빌드가 되었는지 확인합니다. 삭제해도 무방합니다.
- files: 어떤 파일들을 보낼껀지 결정합니다.
  - \*.jar: 스프링 빌드 파일
  - appspec.yml: AWS CodeDeploy 배포용 yml
  - beforeInstall.sh, deploy.sh: appspec에 명시된 쉘스크립트 파일입니다.
- cache: paths: 캐시파일 위치입니다.

여기까지 설정하면 AWS CodeBuild를 통하여 빌드하고 S3로 build.zip파일이 업로드가 됩니다.

![image](https://user-images.githubusercontent.com/49144662/112753422-8fb8dc00-9012-11eb-8656-a39900964ea3.png)

# 3. Github Actions 설정하기

![image](https://user-images.githubusercontent.com/49144662/112753430-96dfea00-9012-11eb-829d-b2b4067f3600.png)

빌드할 Repository에서 Actions를 선택합니다.

![image](https://user-images.githubusercontent.com/49144662/112753435-9e9f8e80-9012-11eb-9791-c0ffc2a7c183.png)

.github/workflows/main.yml

```yaml
name: Actions 이름

on:
  push:
    branches: [master]

jobs:
  buildAndTest:
    name: Github action CD
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
        run: aws deploy create-deployment --application-name cicdtest --deployment-config-name CodeDeployDefault.OneAtATime --deployment-group-name cicd-bluegreen --s3-location bucket=cicdtestsunny,bundleType=zip,key=build.zip
```

```yaml
name: Actions 이름

on:
  push:
    branches: [master]

jobs:
  buildAndTest:
    name: Github action CD
    runs-on: ubuntu-18.04
    defaults:
      run:
        shell: bash
```

- on: push: push냐 Pull Request냐 설정이 가능합니다.
- branches: 브랜치 선택
- buildAndTest: 대상을 지정하는 것으로 다른 이름(ex: backend 등등)으로 바꿀 수 있습니다.
- runs-on: docker 이미지 환경

```yaml
steps:
  - name: 체크아웃 Github-Action
    uses: actions/checkout@v2
```

이 코드를 빼면 Actions가 작동하지 않기 때문에 꼭 추가합니다.

```yaml
- name: AWS 설정
  uses: aws-actions/configure-aws-credentials@v1
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-northeast-2
```

IAM의 Access Key ID, Secret Access Key를 Github의 settings에 설정합니다.

![image](https://user-images.githubusercontent.com/49144662/112753455-a828f680-9012-11eb-8292-fc9d45a35a08.png)

```yaml
- name: CodeBuild 실행
  uses: aws-actions/aws-codebuild-run-build@v1.0.3
  with:
    project-name: cicdtest
```

코드 빌드 실행합니다.

- project-name: AWS CodeBuild에서 생성한 프로젝트 이름을 적습니다.

```yaml
- name: Code Deploy
  run: aws deploy create-deployment --application-name cicdtest --deployment-config-name CodeDeployDefault.OneAtATime --deployment-group-name cicd-bluegreen --s3-location bucket=cicdtestsunny,bundleType=zip,key=build.zip
```

이 내용은 다음에 설명할 AWS CodeDeploy 설정입니다.

여기까지 했으면 Github Actions 설정은 마쳤습니다.

다음은 AWS CodeDeploy 이용한 Blue/Green 무중단 배포를 설정해보겠습니다!

# Trouble Shooting

CodeBuild에서 해당 에러가 났을 경우

![image](https://user-images.githubusercontent.com/49144662/112753515-c0007a80-9012-11eb-94b6-41c64b29a245.png)

![image](https://user-images.githubusercontent.com/49144662/112753556-ddcddf80-9012-11eb-9182-d656e9d3a54d.png)

브랜치를 지정하지 않아서 생긴 에러입니다. 브랜치를 꼭 적을 것!

### 출처

[aws-actions/aws-codebuild-run-build](https://github.com/aws-actions/aws-codebuild-run-build)

[AWS CodeBuild을 Amazon Virtual Private Cloud와 함께 사용](https://docs.aws.amazon.com/ko_kr/codebuild/latest/userguide/vpc-support.html)
