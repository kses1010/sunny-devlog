---
title: 'AWS Chatbot + Slack 이용하여 CI/CD 알림 설정하기'
date: 2021-12-01
category: 'AWS'
draft: false
---

AWS CI/CD 설정은 모두 마쳤다면 이젠 그 결과물을 Slack 알림으로 확인해봅시다!!

# 0. Slack 알람 아키텍처

![image](https://user-images.githubusercontent.com/49144662/144176890-ce8ff03c-bfe6-417c-b343-e887005985d4.png)


# 1. Github Actions에 Slack설정하기

action-slack을 활용하여 Slack 알람을 설정합니다.

```yaml
			- name: action-slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{job.status}}
          fields: repo, ref, workflow, message, author
          author_name: Github Actions
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```

- secrets는 Github Repo에서 settings에서 설정합니다.

> 출처: [action-slack](https://action-slack.netlify.app/)

# 1. IAM 추가하기

IAM에서 설정을 추가합니다. (Amazon SNS, AWS Chatbot)

AWS Chatbot은 권한 설정에 Chatbot 정책이 없기 때문에 정책을 인라인으로 추가합니다.

![image](https://user-images.githubusercontent.com/49144662/144176934-b8c44807-3b82-4ca3-89e0-41594053babb.png)

![image](https://user-images.githubusercontent.com/49144662/144176947-85b4796d-b0b8-4958-9ac3-4b6032c4dc11.png)

- Chatbot을 추가하고 Review policy를 눌러 정책을 생성합니다.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "chatbot:*",
            "Resource": "*"
        }
    ]
}
```

- 또는 JSON 편집기를 사용하여 chatbot의 권한을 생성합니다.

# 2. Amazon SNS, AWS Chatbot 설정하기

## 2.1 Amazon SNS 생성하기

![image](https://user-images.githubusercontent.com/49144662/144176960-a2ccb002-d0c1-4124-a29e-e58c74f73759.png)

- Standard를 선택합니다.

## 2.2 AWS Chatbot 생성하기

![image](https://user-images.githubusercontent.com/49144662/144176985-30385d66-ba73-4b6a-a4a1-c4f2e0e0457d.png)

![image](https://user-images.githubusercontent.com/49144662/144177008-65d0037d-968e-45a0-9b9e-fef62a74472e.png)

- Slack 선택

![image](https://user-images.githubusercontent.com/49144662/144177035-905f111a-e9d2-4be0-9c81-4b261b0158da.png)
허용하고 Slack workspace에서 설정을 합니다.

![image](https://user-images.githubusercontent.com/49144662/144177828-fe968ffd-cca0-4040-a36d-7db7111d53ae.png)
- name: 봇 이름
- Slack channel: 원하는 슬랙 채널을 지정합니다.

![image](https://user-images.githubusercontent.com/49144662/144177097-97bbe1b5-9fe7-4c2b-ac29-b27245986a69.png)
- Permissions: AWS Chatbot을 이용하여 aws cli를 호출할 수 있는 권한을 설정합니다.
  - 저는 Lambda Role을 부여하여 Lambda 호출할 수 있도록 설정했습니다.
- Notifications: 이벤트 및 경보 알림을 전송할 수 있도록 리전과 topic(주제)를 선택합니다.
  - Topics: 이전에 설정했던 Amazon SNS를 선택합니다.

![image](https://user-images.githubusercontent.com/49144662/144177124-152f89b9-f876-491b-9343-099ca7161931.png)
- Amazon SNS에서 생성한 Topics를 확인하시면 Subscriptions에 HTTP 프로토콜로 Chatbot이 추가된 걸 확인할 수 있습니다.

# 3. CodeBuild, CodeDeploy 알람 설정하기

## 3.1 AWS CodeBuild 알람 설정

![image](https://user-images.githubusercontent.com/49144662/144177157-42cd2348-d1c2-4043-8020-c575ed90f876.png)
- Events that trigger notifications: 빌드 실패했을 경우에만 알람설정을 합니다.
- Targets
  - AWS Chatbot 설정으로 합니다.

![image](https://user-images.githubusercontent.com/49144662/144177191-4bf965b5-65fb-41a0-a177-6a0680fbbc9e.png)

- Build에서 알람이 생성되었습니다.

## 3.2 CodeDeploy 알람 생성

![image](https://user-images.githubusercontent.com/49144662/144177214-625f43b8-4e78-438e-aa0c-ce876391176a.png)
- Events that trigger notifications: 성공, 실패만 알려주도록 설정합니다.

# 4. Slack AWS Bot 연결

Amazon SNS에서 설정한 Slack 채널에서 AWS 봇을 다음을 채팅을 쳐서 부릅니다.

- `/invite @aws`

![image](https://user-images.githubusercontent.com/49144662/144177237-d6671653-b858-4e58-b7fd-41d4f24dc35a.png)

# 테스트하기

이제 Github Actions을 이용하여 배포 성공 테스트를 해봅시다.

![image](https://user-images.githubusercontent.com/49144662/144177263-021b53a8-d292-4a4c-acb0-a1ba9ba66b54.png)
성공알림 테스트가 잘되었군요. 

이번에는 빌드 실패 테스트를 합시다.

![image](https://user-images.githubusercontent.com/49144662/144177289-fad9bdea-716e-4754-a896-c4e58ade8610.png)

배포 실패 테스트
![image](https://user-images.githubusercontent.com/49144662/144177310-2beb23e7-d24f-4573-a2bc-5578b7fb798b.png)

실패알림 테스트도 잘되는군요!!

여기까지 Pre Delivery(Github) → Post Build(CodeBuild) → Post Deploy(CodeDeploy) 알람 설정을 모두 마쳤습니다.

알람 설정을 삭제하고 싶을 때는 CodeStar Notification 권한이 필요하니 꼭 IAM에서 설정하세요!!