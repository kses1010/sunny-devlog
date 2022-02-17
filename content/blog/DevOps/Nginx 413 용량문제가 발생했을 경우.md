---
title: 'Ngixn 413 용량문제 해결하기'
date: 2022-02-17
category: 'DevOps'
draft: false
---

# 0. 문제 발생

![image](https://user-images.githubusercontent.com/49144662/154433251-804e82b6-3b81-4249-afaa-f25a6dc9d7af.png)

파일이 업로드 하던중에 용량문제로 nginx에서 413 코드가 나왔습니다.

## Nginx 설정하기

기존의 nginx 설정하지 않으면 업로드 용량을 1MB로 잡습니다.

nginx설정으로 들어가서 수정해봅시다.

```bash
cd /etc/nginx
sudo vi nginx.conf
```

nginx.conf

```bash
http {

	... nginx 세팅 어쩌구...

	## Set Client upload Size
	client_max_body_size 100M;

}
```

- `client_max_body_size` : 업로드 용량을 결정할 수 있습니다. (없으면 직접 추가할 것)

이후 nginx를 다시 실행합니다.

```bash
sudo service nginx restart
```

### 출처

[nginx 413 Request Entity Too Large 에러 해결하기, 파일 업로드 사이즈](https://webisfree.com/2018-03-29/nginx-413-request-entity-too-large-에러-해결하기-파일-업로드-사이즈)