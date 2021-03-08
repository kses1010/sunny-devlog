---
title: 'Github Actions, AWS Deploy로 CI/CD해보자!'
date: 2021-01-11
category: 'CI/CD'
draft: false
---

# Github Action으로 Spring boot, React.js를 배포해보자!

![image](https://user-images.githubusercontent.com/49144662/104315729-bd67ac00-551e-11eb-84f1-6144ec8fc980.png)

이 문서는 Github Actions을 이용하여 CI/CD 환경을 만들고 EC2에 Spring boot, React.js를 배포하는 것이 목표입니다.

## 0. Github Actions란?

**Github Actions**는 Github 저장소를 기반으로 Github에서 제공하는 Workflow 자동화 도구 입니다. Workflow는 Github 저장소에서 발생하는 `build`, `test`, `release`, `deploy` 등 다양한 이벤트를 기반으로 직접 원하는 Workflow를 만들 수 있습니다.

Github Actions는 Runner 위에서 실행이 되고, Runner는 가상 머신 위에서 실행이 됩니다. Github에서 두 종류의 Runner를 제공합니다. Github-hosted Runner, Self-Hosted Runner(사용자가 직접 호스팅하는 환경)가 있습니다.

### 스펙

Github-Hosted Runner는 마소에서 지원해주는 Azure의 가상 머신 위에서 돌아갑니다. 하지만 가상 머신의 사양은 그리 좋진 않습니다.

- 2-core CPU
- 7GB of RAM memory
- 14 GB of SSD disk space

백엔드 저장소 빌드 보다는 프론트엔드 저장소 빌드하는 목적으로 둔다면 충분히 쓸 수 있을 듯하네요.

### 비용

공개 저장소 경우에는 무료이며, 비공개 저장소는 계정에 부여한 무료 사용량 이후에 과금이 됩니다. Workflow는 저장소마다 최대 20개까지 등록이 가능하며 Workflow의 Job이라는 단계마다 최대 6시간 동안 실행이 가능하며, 초과하면 자동으로 중지됩니다.

그리고 Job 안에서 Github API를 호출할 때 1시간 동안 최대 1000번까지만 가능합니다.

### 그런데 왜 Github Actions?

비용적인 장점도 좋지만 다른 CI/CD 툴(Circle CI/CD, Travis CI/CD)보다 Github Actions를 사용한 이유는

1. 다른 CI/CD 보다 굉장히 쉽고 간단합니다. 공식문서와 최근에는 국내에도 여러 레퍼런스가 있어 참고하기 쉽습니다.
2. 해당 프로젝트 저장소에서 따로 CI/CD 툴을 설치하거나 hook을 걸 필요가 없습니다.
3. Travis CI/CD 보다 훨씬 빨랐습니다. (제경우에는...)

사기업 대부분은 Jenkins를 사용할 테지만 간단, 작은 프로젝트 같은 경우에는 Github Actions가 잘 맞았습니다.

그럼 AWS CodeDeploy설정부터 차례차례 진행겠습니다~ :smile:

## 1. EC2 설정 (Ubuntu 18.04)

### Ubuntu 18.04 에서 Directory not empty @ dir_s_rmdir 에러

- CodeDeploy agent는 Ruby로 작동하기 떄문에 ruby 설치는 필수!
- [Ubuntu 18.04](http://releases.ubuntu.com/18.04/) 에서는 배포된 파일들을 보관할 폴더가 비어있지 않다면 `Directory not empty @ dir_s_rmdir` 에러가 발생합니다.
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

```bash
# 자바 설치
sudo apt install openjdk-8-jdk
java -version

# codeDeploy 설치
sudo apt update
# Ubuntu 18.04 제외한 OS
sudo apt install ruby
sudo apt install wget

cd /home/ubuntu
wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto

# codeDeploy 확인
sudo service codedeploy-agent status
# 멈추면
sudo service codedeploy-agent start
```

`wget [https://버켓네임.s3.리전식별자.amazonaws.com/latest/install](https://bucket-name.s3.region-identifier.amazonaws.com/latest/install)`

CodeDeploy 리소스 키트 버킷 참조 할 것: [링크](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/resource-kit.html#resource-kit-bucket-names)

## 2.1 IAM설정하기

- AWS에서 EC2의 IAM 역할 설정을 CodeDeployFullAccess, S3를 추가합니다.

```
AWSCodeDeployFullAccess
AWSS3FullAccess
```

![01](https://user-images.githubusercontent.com/49144662/104314750-3534d700-551d-11eb-99bc-1f2b141e2a1c.png)

![02](https://user-images.githubusercontent.com/49144662/104314754-382fc780-551d-11eb-8b3b-7261f7a1cc61.png)

- CodeDeploy의 IAM 역할 설정: CodeDeployFullAccess 추가

## 2.2 IAM 역할 EC2

![03](https://user-images.githubusercontent.com/49144662/104314757-3960f480-551d-11eb-97e6-d29cc29d0966.png)

![04](https://user-images.githubusercontent.com/49144662/104314759-3960f480-551d-11eb-979f-ab30fd86e4fe.png)

## 3.1 CodeDeploy 설정

- CodeDeploy 애플리케이션 생성

![05](https://user-images.githubusercontent.com/49144662/104314761-39f98b00-551d-11eb-8960-b5f88734666c.png)

- 컴퓨터 유형: EC2/온프레미스

  애플리케이션 이름: 꼭 기억해야합니다!! Github Action workflow 사항에 기입해야하기 떄문이죠.

![06](https://user-images.githubusercontent.com/49144662/104314762-39f98b00-551d-11eb-9be0-c936304e3bcd.png)

## 3.2 배포 그룹 생성

- 배포 그룹 이름: 여기도 기억해야할 대상. workflow 기입대상입니다.
- 서비스 역할 설정: CodeDeploy역할 선택

![07](https://user-images.githubusercontent.com/49144662/104314765-3a922180-551d-11eb-9e0c-1afb09e77ffd.png)

- 배포방법 선택: 현재위치
- 환경구성
  - Amazon EC2 인스턴스
  - 키-값: EC2인스턴스 이름보고 선택

![08](https://user-images.githubusercontent.com/49144662/104314766-3b2ab800-551d-11eb-82d0-aa51ec5e5d51.png)

        EC2의 이름 태그 확인하고 연결하기

![09](https://user-images.githubusercontent.com/49144662/104314768-3b2ab800-551d-11eb-89f7-6158e1c0532c.png)

- 배포 설정: CodeDeployDefault.AllAtOnce 선택
  - 로드 밸런싱 비활성화 (설정하지 않았기 때문에 선택하지 않겠습니다)

![10](https://user-images.githubusercontent.com/49144662/104314769-3bc34e80-551d-11eb-9e96-48c3a31d96d7.png)

## 4. 액세스 키 설정

- 액세스 키 ID / 액세스 Secret Key

![11](https://user-images.githubusercontent.com/49144662/104314772-3bc34e80-551d-11eb-8543-1e0bd3d25cf3.png)

액세스 키 ID와 액세스 Secret Key는 Github Action의 Secret으로 사용되니 잃어버리지 말 것!!

## 5. workflow 설정

고생하셨습니다 ~ 여기까지 하면 AWS환경 세팅은 끝났습니다.

workflow 설정시 제일 중요한건 actions/checkout 을 꼭 지정해야합니다.

하지 않는다면 해당 working-directory를 읽지 못하고 그냥 종료되기 때문에 꼭 적어야 합니다. 이것 떄문에 저도 몇시간 버린 기억이 납니다 :cry:

### CI 설정

```yaml
name: CI

on:
  pull_request:
    branches: [dev]

jobs:
  backend:
    name: CI with Gradle
    runs-on: ubuntu-18.04
    defaults:
      run:
        shell: bash
        working-directory: BE

    steps:
      - name: 체크아웃 Github-Action
        uses: actions/checkout@v2

      - name: 자바 JDK 1.8 설치
        uses: actions/setup-java@v1
        with:
          java-version: 1.8

      - name: gradlew 권한 부여
        run: chmod +x ./gradlew

      - name: Gradle 빌드
        run: ./gradlew build

  frontend:
    name: CI with Node.js
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        working-directory: client

    strategy:
      matrix:
        node-version: [12.16.x]

    steps:
      - name: 체크아웃 Github-Action
        uses: actions/checkout@v2

      - name: node.js 12 설치 ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}

      - name: npm 설치
        run: npm install

      - name: npm 빌드
        run: npm run build
```

- 여기서 한 스크립트에 두개를 CI로 돌리려면 이런식으로 구분을 지어야 합니다.

```yaml
jobs:
  작업이름1:

### 해야할 runs on, steps 등등

작업이름2:
```

### CD

- CD는 프론트엔드와 백엔드의 CodeDeploy를 돌리는 방식을 각각 두개의 CodeDeploy를 만들어 사용했습니다.

```yaml
name: CD with Gradle

on:
  push:
    branches: [dev]

jobs:
  backend:
    name: CD with Gradle
    runs-on: ubuntu-18.04
    defaults:
      run:
        shell: bash
        working-directory: BE

    steps:
      - name: 체크아웃 Github-Action
        uses: actions/checkout@v2

      - name: 자바 JDK 1.8 설치
        uses: actions/setup-java@v1
        with:
          java-version: 1.8

      - name: gradlew 권한 부여
        run: chmod +x ./gradlew

      - name: Gradle 빌드
        run: ./gradlew bootJar

      - name: AWS 설정
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2

      - name: S3 업로드
        run: aws deploy push --application-name EC2-S3-Deploy --description "This is a revision for the application airbnb_App" --s3-location s3://codesquad-deploy/server/build.zip --source .

      - name: Code Deploy
        run: aws deploy create-deployment --application-name EC2-S3-Deploy --deployment-config-name CodeDeployDefault.OneAtATime --deployment-group-name dev --s3-location bucket=codesquad-deploy,bundleType=zip,key=server/build.zip
```

```yaml
name: CD with Node

on:
  push:
    branches: [dev]

jobs:
  backend:
    name: CD with Node
    runs-on: ubuntu-18.04
    defaults:
      run:
        shell: bash
        working-directory: client

    strategy:
      matrix:
        node-version: [12.16.x]

    steps:
      - name: 체크아웃 Github-Action
        uses: actions/checkout@v2

      - name: node.js 12 설치 ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}

      - name: npm 설치
        run: npm install

      - name: npm 빌드
        run: npm run build

      - name: zip 생성
        run: zip -qq -r ./build-fe.zip .
        shell: bash

      - name: AWS 설정
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: S3 업로드
        run: aws s3 cp --region ap-northeast-2 ./build-fe.zip s3://codesquad-deploy/client/build-fe.zip

      - name: Code Deploy
        run: aws deploy create-deployment --application-name EC2-S3-Deploy-FE --deployment-config-name CodeDeployDefault.OneAtATime --deployment-group-name dev-FE --s3-location bucket=codesquad-deploy,bundleType=zip,key=client/build-fe.zip
```

프론트 CD 같은경우에는 zip 생성하여 cp로 한 이유는

`zip does not support timestamps before 1980` 에러가 나서 Action이 작동이 안되기 때문에 zip를 직접하여 cp로 S3 복사했습니다. 만약 에러가 발생하지 않으면 그대로 진행하셔도 됩니다.

- AWS 연결 설정

```yaml
- name: AWS 설정
  uses: aws-actions/configure-aws-credentials@v1
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-northeast-2
```

4번에서 받았던 액세스키 ID와 액세스 Secret Key를 적어주면 됩니다.

적용시키려는 Github Repository에서

![12](https://user-images.githubusercontent.com/49144662/104314776-3c5be500-551d-11eb-87d9-2c1cb700086c.png)

![13](https://user-images.githubusercontent.com/49144662/104314779-3cf47b80-551d-11eb-94a6-ad038fcceb1a.png)

![14](https://user-images.githubusercontent.com/49144662/104314780-3cf47b80-551d-11eb-8f5e-5bff8e56c7da.png)

액세스 ID 키와 액세스 Secret을 설정하면 됩니다. 그러나, 만약 권한이 없을 경우 권한이 있는분에게 설정해달라고 부탁합니다.

- S3 경로와 배포 경로

```yaml
	    - name: S3 업로드
        run: aws deploy push --application-name EC2-S3-Deploy --description "This is a revision for the application airbnb_App" --s3-location s3://codesquad-deploy/server/build.zip --source .

      - name: Code Deploy
        run: aws deploy create-deployment --application-name EC2-S3-Deploy --deployment-config-name CodeDeployDefault.OneAtATime --deployment-group-name dev --s3-location bucket=codesquad-deploy,bundleType=zip,key=server/build.zip
```

```yaml
## S3 업로드 (zip로만 파일이 옮기기 때문에 zip로 함)
--application-name: CodeDeploy 애플리케이션 이름 (3번에서 확인하자)
--description: 설명
--s3-location: s3의 경로로 s3://s3이름/파일이름.zip

--deployment-config-name: 배포설정
--deployment-group-name: 배포그룹 이름 (3번에서 확인하자)
--s3-location : bucket=버킷이름, bundleType=파일 타입, key=파일 이름

key는 build.zip가 아닌 server/build.zip로 설정해야한다.

```

## 6. deploy 설정

### ./gradlew 파일 디렉토리에 세팅하기

- appspec.yml

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /home/ubuntu/myapp(폴더이름은 아무거나)
permissions:
  - object: /home/ubuntu/myapp
    owner: ubuntu
    group: ubuntu
    mode: 755

hooks:
  AfterInstall:
    - location: deploy.sh
      timeout: 60
      runas: root
```

appspec.yml 파일을 생성하여 어디에 풀 건지 지정하고 hooks에서 시작할 쉘스크립트를 정합니다.

혹시 몰라서 EC2에서 해당 폴더를 생성했습니다.

- deploy.sh

```bash
#!/usr/bin/env bash

REPOSITORY=/home/ubuntu/myapp
cd $REPOSITORY

APP_NAME=airbnb
JAR_NAME=$(ls $REPOSITORY/build/libs/ | grep '.jar' | tail -n 1)
JAR_PATH=$REPOSITORY/build/libs/$JAR_NAME

CURRENT_PID=$(pgrep -f $APP_NAME)

if [ -z $CURRENT_PID ]
then
  echo "> 종료할것 없음."
else
  echo "> kill -9 $CURRENT_PID"
  kill -9 $CURRENT_PID
  sleep 5
fi

echo "> $JAR_PATH 배포"
nohup java -jar -Dspring.profiles.active=prod $JAR_PATH > /dev/null 2> /dev/null < /dev/null &
```

## 프론트 배포

- appspec.yml

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /home/ubuntu/deploy-fe

permissions:
  - object: /home/ubuntu/deploy-fe
    owner: ubuntu
    group: ubuntu
    mode: 755

hooks:
  AfterInstall:
    - location: deploy-fe.sh
      timeout: 60
      runas: root
```

- deploy.sh

```bash
#!/usr/bin/env bash

echo "> FE 배포"
sudo cp -rf /home/ubuntu/deploy-fe/dist/* /var/www/html
```

## AS

- 프론트 빌드해서 빌드 실패가 나는 경우

```bash
0 info it worked if it ends with ok
1 verbose cli [
1 verbose cli   '/opt/hostedtoolcache/node/12.18.0/x64/bin/node',
1 verbose cli   '/opt/hostedtoolcache/node/12.18.0/x64/bin/npm',
1 verbose cli   'run',
1 verbose cli   'build'
1 verbose cli ]
2 info using npm@6.14.4
3 info using node@v12.18.0
4 verbose run-script [ 'prebuild', 'build', 'postbuild' ]
5 info lifecycle client@0.1.0~prebuild: client@0.1.0
6 info lifecycle client@0.1.0~build: client@0.1.0
7 verbose lifecycle client@0.1.0~build: unsafe-perm in lifecycle true
8 verbose lifecycle client@0.1.0~build: PATH: /opt/hostedtoolcache/node/12.18.0/x64/lib/node_modules/npm/node_modules/npm-lifecycle/node-gyp-bin:/home/runner/work/issue-tracker-03/issue-tracker-03/client/node_modules/.bin:/opt/hostedtoolcache/node/12.18.0/x64/bin:/usr/share/rust/.cargo/bin:/home/runner/.config/composer/vendor/bin:/home/runner/.dotnet/tools:/snap/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:/home/runner/.dotnet/tools
9 verbose lifecycle client@0.1.0~build: CWD: /home/runner/work/issue-tracker-03/issue-tracker-03/client
10 silly lifecycle client@0.1.0~build: Args: [ '-c', 'node scripts/build.js' ]
11 silly lifecycle client@0.1.0~build: Returned: code: 1  signal: null
12 info lifecycle client@0.1.0~build: Failed to exec build script
13 verbose stack Error: client@0.1.0 build: `node scripts/build.js`
13 verbose stack Exit status 1
13 verbose stack     at EventEmitter.<anonymous> (/opt/hostedtoolcache/node/12.18.0/x64/lib/node_modules/npm/node_modules/npm-lifecycle/index.js:332:16)
13 verbose stack     at EventEmitter.emit (events.js:315:20)
13 verbose stack     at ChildProcess.<anonymous> (/opt/hostedtoolcache/node/12.18.0/x64/lib/node_modules/npm/node_modules/npm-lifecycle/lib/spawn.js:55:14)
13 verbose stack     at ChildProcess.emit (events.js:315:20)
13 verbose stack     at maybeClose (internal/child_process.js:1021:16)
13 verbose stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:286:5)
14 verbose pkgid client@0.1.0
15 verbose cwd /home/runner/work/issue-tracker-03/issue-tracker-03/client
16 verbose Linux 5.3.0-1028-azure
17 verbose argv "/opt/hostedtoolcache/node/12.18.0/x64/bin/node" "/opt/hostedtoolcache/node/12.18.0/x64/bin/npm" "run" "build"
18 verbose node v12.18.0
19 verbose npm  v6.14.4
20 error code ELIFECYCLE
21 error errno 1
22 error client@0.1.0 build: `node scripts/build.js`
22 error Exit status 1
23 error Failed at the client@0.1.0 build script.
23 error This is probably not a problem with npm. There is likely additional logging output above.
24 verbose exit [ 1, true ]
```

- 이 문구를 확인해야합니다.

```
Treating warnings as errors because process.env.CI = true.
Most CI servers set it automatically.
```

이 문구는 CI 설정이 안되어있어서 워닝도 에러로 판단하겠다라는 뜻.

추가적으로 이걸 설정 하면 이상없습니다.

```yaml
run: unset CI && npm run build
```

## ETC

- 실행하고 나서 Actions을 보고 기도하고, CodeDeploy에 가서 잘되는지 기도했던 기억이 납니다.
- 가끔 전부 다 초록불이 들어왔는데 배포가 안되어있다면 쉘스크립트가 잘못 만들어서 그런 것입니다. 그럴때에는 로그를 보면서 하나씩 고쳐야 합니다.
- 그 이후엔 다행히 저는 정상적으로 작동되었습니다. 이젠 더이상 EC2에서 git pull -> build할 필요가 없어졌습니다.

Github Actions과 AWS CodeDeploy를 이용하여 쾌적한 개발활동하길 바랍니다 :yum:

## 출처

- 코드스쿼드 에어비앤비 11조 (Dion, Han): [링크](https://github.com/codesquad-member-2020/airbnb-11)
- [github action과 aws code deploy를 이용하여 spring boot 배포하기](<https://isntyet.github.io/deploy/github-action%EA%B3%BC-aws-code-deploy%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-spring-boot-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0(1)/>)
- [AWS EC2 / AWS S3 / Codeship 으로 웹 서비스 테스트 및 자동 배포](https://hyungseok.kr/developments/11#10-ubuntu-1804-%EC%97%90%EC%84%9C-directory-not-empty-dir_s_rmdir-%EC%97%90%EB%9F%AC)
- [https://johngrib.github.io/wiki/CodeDeploy/](https://johngrib.github.io/wiki/CodeDeploy/)
