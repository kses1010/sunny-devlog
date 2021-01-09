---
title: 'HTTPS Let''s Encrypt 설정하기(EC2에 설정하기)'
date: 2021-01-09
category: 'AWS'
draft: false
---

# 1. Let's Encrypt

Let's Encrypt는 무료 [SSL / TLS 인증서](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs)를 얻고 설치할 수 있는 인증 기관으로, 웹 서버에서 암호화 된 HTTPS를 사용할 수 있게 해줍니다. 또한, Certbot 이라는 자동화 클라이언트를 제공하여 Apache 및 Nginx에서 인증서를 획득하고, 설치하는 전체 프로세스가 자동화 되어 있습니다.

이 도움말에서는 Certbot을 사용하여 Ubuntu 18.04 에서 Nginx용 무료 SSL 인증서를 받고, 자동으로 갱신하도록 설정할 수 있습니다.

또한, 이 도움말에서는 default 파일 대신, 별도의 Nginx 서버 블록 파일을 사용합니다. 일반적인 실수를 방지하고, default 파일을 예외 처리 파일로 유지할 수 있으므로, 도메인마다 새로운 Nginx 서버 블록 파일을 만드는 것을 추천합니다.

# 2. 서버 블록 설정하기

### Nginx 설치

```bash
sudo apt update
sudo apt install nginx
```

### 방화벽 설치(AWS 쓰고있으면 Skip)

### 웹서버 확인

```bash
sudo service nginx status

nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: en
   Active: active (running) since Wed 2020-09-16 17:19:24 KST; 1h 19min ago
     Docs: man:nginx(8)
  Process: 7597 ExecStop=/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 -
  Process: 7611 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code
  Process: 7599 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process
 Main PID: 7615 (nginx)
    Tasks: 2 (limit: 1140)
   CGroup: /system.slice/nginx.service
           ├─7615 nginx: master process /usr/sbin/nginx -g daemon on; master_pro
           └─7754 nginx: worker process
```

### 서버 블록 설정하기

우분투 18.04에서 Nginx설치하면 기본으로 `/var/www/html` 을 제공합니다. 새로운 Nginx 서버블록을 만드려면 자신의 도메인이름으로 된 폴더를 만들어야 합니다.

```bash
sudo mkdir -p /var/www/{도메인이름}/html
sudo chown -R $USER:$USER /var/www/{도메인이름}/html
sudo chmod -R 755 /var/www/{도메인이름}
```

이제부터 {도메인이름}은 `example.com` 으로 지정하겠습니다.

지정하고 나면 `/etc/nginx/sites-available/` 에 `example.com` 을 만듭니다.

```bash
sudo vi /etc/nginx/sites-available/example.com

server {
        listen 80;
        listen [::]:80;

        root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

				location / {
								#try_files $uri $uri/ =404; 주석처리
                proxy_pass http://localhost:8080;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }
}
```

proxy_pass를 추가하여 리버스프록시 설정합니다. 

이 파일을 `sites-enabled` 폴더에 연결합니다.

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```

 `/etc/nginx/nginx.conf` 에서 해시 버킷 메모리 문제 해결하기

```bash
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...
```

Nginx 설정이 잘됐는지 확인하기

```bash
sudo nginx -t

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

그 밑 문구처럼 뜬다면 바로 nginx restart 합니다.

```bash
sudo service nginx restart
```

# 3. Certbot 설치

Let's Encrypt를 사용하여 SSL 인증서를 얻기 위해서는 우분투 서버에 Certbot 클라이언트를 설치해야 합니다. Certbot의 개발은 빠르게 개발되고 있어서, 우분투 `apt` 에서 제공하는 Certbot 패키지는 조금 오래된 버젼입니다. 하지만, Certbot의 레포지토리는 항상 최신을 유지하고 있습니다. 우리는 레포지토리를 사용하여 설치할 예정입니다.

아래 명령어를 추가하여 레포지토리를 추가합니다.

```bash
sudo add-apt-repository ppa:certbot/certbot
```

설치하기

```bash
sudo apt install python-certbot-nginx
```

# 4. SSL 인증서 가져오기

Certbot은 다양한 웹 서버에 대하여 SSL 인증서를 얻을 수 있는 플러그인을 제공합니다.

Nginx 플러그인을 사용하기 위해, 다음과 같이 입력합니다.

```bash
sudo certbot --nginx -d example.com
```

certbot은 -d 우측에 있는 도메인들을 유효하게 만드는 설정을 진행할 수 있습니다.

진행시 다음처럼 뜹니다.

```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): 이메일 주소

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: 

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: 
```

이메일 주소 입력하고, 서비스 약관에 동의해야하는 절차를 수행하면 됩니다.

위 절차가 성공적으로 진행한다면, certbot은 https를 어떻게 설정할 지 묻습니다.

```bash
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for example.com
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/example.com

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
```

질문: http 트래픽을 https로 리다이렉트 할 것입니까? 아니면 http 접근을 막으실 겁니까?

1번: 리다이렉트 없음 - 웹서버 설정에 아무런 변화도 없습니다.

2번: 리다이렉트 - 모든 http 연결을 https로 리다이렉트 합니다. 웹서버 설정을 바꿀 수 있습니다. 바뀐 설정은 다시 되돌릴 수 있습니다.

여기서 2번을 선택하여 http연결을 https로 리다이렉트합니다.

```bash
Congratulations! You have successfully enabled
https://example.com

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=example.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2020-12-15. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Let's Encrypt의 인증서는 설치가 완료된 이후 90일 까지만 유효합니다. 

90일 지나기전에 certbot을 다시 실행하여 인증서를 갱신해야합니다.

# 5. SSL 자동으로 갱신시키기

Let's Encrypt의 인증서는 90일 동안만 유효합니다. 인증서를 자주 갱신시키는 것이 권장되기 때문입니다. 우리는 certbot 패키지를 설치하여 이 문제를 자동화 시킬 수 있습니다. `/etc/cron.d` 내의 스크립트는 하루 두번씩 실행되어 만료일까지 30일 이내의 모든 인증서를 자동으로 갱신 시킬 수 있습니다.

갱신 절차를 테스트 해보려면, 다음 명령어를 입력하세요.

```bash
$ sudo certbot renew --dry-run
```

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/example.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator nginx, Installer nginx
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for example.com
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of nginx server; fullchain is
/etc/letsencrypt/live/example.com/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/example.com/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
```

오류가 없다면, 당신은 모든 설정을 마친 것입니다. 이제 certbot이 자동적으로 인증서를 갱신하고, Nginx를 재시작 할것입니다. 만약 자동 갱신 프로세스가 실패할 경우, 인증서가 만료되기 이전에 사전에 지정된 이메일로 경고 메시지가 보내질 것입니다.

실제로 수동으로 갱신하려면

```bash
sudo certbot renew
```

실행하면 됩니다.

## 출처

- [https://velog.io/@pinot/Ubuntu-18.04](https://velog.io/@pinot/Ubuntu-18.04%EC%97%90%EC%84%9C-Lets-Encrypt%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-Nginx%EC%97%90-SSL%EC%9D%84-%EC%A0%81%EC%9A%A9%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
- [서버 블록 설정하기](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04#step-5-setting-up-server-blocks-(recommended))