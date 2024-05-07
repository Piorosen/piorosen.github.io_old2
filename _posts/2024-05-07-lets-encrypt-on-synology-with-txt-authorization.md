---
layout: post
title: Let's Encrypt를 이용하여 시놀로지에 SSL 적용 (w/ Certbot)
author: piorosen
categories: [Blogging, Develop]
tags: [lets-encrypt,synology,ssl,cloudflare,txt,certbot]
hide_title: false
---

# 개요

우선 본 내용은 필자가 까먹거나 다시 리마인드용으로 남기기 위해서, 또한 한번 더 정리해서 기록할 겸 작성한 포스팅이므로, 원글을 확인하고자 한다면 [[Here]](https://velog.io/@atmost1815/Synology%EC%97%90%EC%84%9C-Wildcard-SSL-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EC%9E%90%EB%8F%99-%EA%B0%B1%EC%8B%A0%ED%95%98%EA%B8%B0) 을 참조하길 바란다. 또한, 해당 내용은 Lets Encrypt를 활용하여 SSL 인증서를 발급과 동시에, 와일드 카드 인증서를 발급을 받을 것이다. 또한, 누구나 쉽게 따라할 수 있도록 만들고자 한다. 다만, 기본 선행적인 지식이 필요하다. 

1. Secure SHell (SSH)
2. Docker
3. `Readable` Shell Script
4. `if possible` for DNS Service, Cloudflare

크게 3가지의 개념이 필요하며, 이것만 충분히 가능하다면 쉽게 해당 글을 따라하여 `Lets Encrypt`의 인증서를 발급받고, 사용할 수 있을것이라 예상이 된다. 원글의 작성자의 경우에는 `Shell` 스크립트와 `Docker` 기반으로 작성이 되어져 있기 때문에 처음 사용한다면 익숙하지 않아 어려움을 겪을수가 있다. 따라서 필자는 가능하다면 원형을 훼손시키지 않은채 쉽게 쓰도록 약간의 변형을 거칠 것이다.

# 목차

1. (생략) Install Docker on Synology
2. (생략) Connect to Synology Server via SSH 
3. 자동화를 작업하기 위한 작업 영역 세팅
4. Let's Encrypt을 위한 도메인 주소 소유 인증 절차
5. 발급한 인증서를 시놀로지 시스템에 적용
6. 자동 갱신 설정


## 3. 자동화를 작업하기 위한 작업 영역 세팅

필자의 경우에는 `main` 이라는 저장소 안에 `ssl` 이라는 폴더를 생성하여 진행하였다. 즉 터미널로 접근한다면 `/volume1/main/ssl/` 이라는 작업 공간으로 잡았다는 것을 의미한다. 터미널로 이용해서 직접 폴더를 생성하여 진행하여도 무관하다. 즉, 아래와 같은 터미널 명령어를 수행하여도 동일한 결과를 나타낸다는 것을 의미한다. 당연하게도, `$` 맨 앞에 나타내는 달러의 의미는 리눅스의 환경에서 터미널 명령어를 수행한다라는 암묵적인 규칙중 하나이다. (정말 쓸모 없는 지식이지만 리눅스에서는 `$`, 유닉스 환경[macOS] 의 경우에는 `%` 이다. 따로 의미는 정말 없다. 그냥 컴퓨터가 처음 나왔을 때, 논문에서 명령어를 직관적으로 구별하기 위해서 나온 것이라 보면 편하다.)

```sh
$ mkdir /volume1/main/ssl
```

![](/assets/img/post/2024-05-07-01.png)

## 4. Let's Encrypt을 위한 도메인 주소 소유 인증 절차

폴더를 생성하였다면, `SSH(터미널)`을 통해 `cd /volume1/main/ssl` 로 작업 영역으로 이동 한다. 그 이후에 `vim`이나 `nano`를 통해 아래의 코드를 추가한다. 파일명은 따로 신경쓰진 않지만, `certbot.sh` 이나 확장자 명이 없어도 무관하다. 편리하게 이름을 작성하면 된다.

생성 한 뒤에 반드시 확인해야하는 내용이 2가지가 있다.
1. `WORK_DIRECTORY` 가 반드시 `/volume1/main/ssl` 인지 확인할 것이다. 현재 기본값인 `$(pwd)`는 현재 디렉토리를 기준으로 하므로, `cd /volume1/main/ssl` 란 명령어를 수행하였다면 생략하여도 무관하다.

2. 현재 소유중인 도메인 주소가 `example.com`이 맞는지 확인한다. 필자의 경우에는 `udon.party`를 소유중이므로, `DOMAIN_URL=udon.party`라 작성하였다. (도메인 주소는 어차피 해킹 대상이나, 숨겨야하는 정보가 아니므로 공개한다., 어차피 블로그의 주소가 `udon.party` 이므로 문제가... 없다.)

그 이후에는 `./certbot.sh` 또는 `./{독자가 생성한 파일명으로}` 으로 실행한다. 만약 오류가 생긴다면, 실행 권한이 부족한 것이므로 `chmod +x {독자가 생성한 파일명}` 또는 `chmod +x certbot.sh`으로 파일에 권한을 부여하고, 다시 실행해 본다.

```sh
#!/bin/bash

CONTAINER_NAME=certbot # 변경하지 않아도 무관한 영역
WORK_DIRECTORY=$(pwd)  # 자동화를 작업하기 위한 작업 영역 세팅할 경로, 그것이 아니라면 현재 디렉토리를 기반으로 수행함.
DOMAIN_URL=example.com # 해당 글을 읽고 있는 사람이 가지고 있는 도메인 주소

docker run -it --rm \
  --pull=always \
  --name=${CONTAINER_NAME} \
  -e "TZ=Asia/Seoul" \
  -v "${WORK_DIRECTORY}/etc:/etc/letsencrypt" \
  -v "${WORK_DIRECTORY}/var:/var/lib/letsencrypt" \
    certbot/certbot certonly \
  -d "${DOMAIN_URL}" \
  -d "*.${DOMAIN_URL}" \
  --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

그 뒤에는 터미널 창에 무언가의 내용이 출력이 되게 되는데, 간단한 영어 해석과 `DNS`에 대한 개념이 있다면 쉽게 따라할 수 있다. 그렇지만 최대한 해석을 해보자면

```sh
# /var/log/letsencrypt/letsencrypt.log 에 로그 파일을 저장함.
Saving debug log to /var/log/letsencrypt/letsencrypt.log
# udon.party 인증 요청
Requesting a certificate for udon.party

-
# DNS에 이름은 _acme-challenge인 TXT 레코드를 추가하세요.
Please deploy a DNS TXT record under the name:
_acme-challenge.udon.party. with the following value:

# 그리고 값을 BAvL0h4rtSmoS21f0zrXJ0PnCc0tVJG9Er0YpIyt931 으로 지정하세요.
BAvL0h4rtSmoS21f0zrXJ0PnCc0tVJG9Er0YpIyt931

# 게속하기 이전에, TXT 레코드가 정상적으로 추가가 되었는지 확인하세요! 이는 DNS Provider가 추가를 해줘야지만 계속할 수 있습니다. 그렇게 때문에 시간이 걸릴 수 있습니다. 그리고 위의 TXT 레코드를 추가하고 정상적으로 배포가 되었는지 확인하기 위해서는 `https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.udon.party.` 에서 확인하세요.
Before continuing, verify the TXT record has been deployed. Depending on the DNS provider, this may take some time, from a few seconds to multiple minutes. You can check if it has finished deploying with aid of online tools, such as the Google Admin Toolbox: https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.on.party.
Look for one
or more bolded line(s) below the line
'¡ANSWER'. It should show the
values) you've just added.
Press Enter to Continue
```

요약하자면, 뭔가 모르는 TXT 레코드를 DNS 서버에 등록해야한다는 의미이다. 그래서 이를 조금 더 쉽게 표현하자면 아래의 그림 순으로 터미널 창에 텍스트가 나타나게 되고, `Cloudflare`에 접속하여 아래와 같이 `Name`과, `Content`에 값을 넣고 추가하면 된다.

그리고, `DNS Provider(=Cloudflare)`에서 추가하더라도, 바로 다음으로 진행하면 안되고, 터미널 창에서 나온 `URL` 주소를 클릭하여 반드시 정상적으로 추가가 되었는지 확인해야한다.

![](/assets/img/post/2024-05-07-02.png)
![](/assets/img/post/2024-05-07-03.png)

이 과정을 총 2번 진행하므로, 2번 진행하면 `/volume1/main/ssl` 폴더에 `etc`와 `var` 폴더가 생성이 되고, `./etc/archive/udon.party` 란 폴더에 4개의 파일이 생성이 되었다면 인증 절차 과정은 모두 끝나게 된 것이다.

```
cert.pem
chain.pem
fullchain.pem
privkey.pem
```

## 5. 발급한 인증서를 시놀로지 시스템에 적용

`/volume1/main/ssl` 폴더를 기준으로 `./etc/archive/udon.party`에 접속한다면, 4개의 파일들 중에서 아래의 3개를 다운로드 받는다.
`cert.pem`, `chain.pem`, `privatekey.pem`을 3개를 다운로드 받는다.
다운로드 받는 방법은 `scp`를 이용하거나, `ftp`를 이용하거나, `dsfile on synology` 를 이용하여 다운로드 받도록 한다. 

그 이후에는, 시놀로지에서 `기본 인증서로 설정` 체크박스를 선택하고, 인증서를 가져오기 하면 된다.
그리고, 개인키는 `private.pem`, 인증서는 `cert.pem`, 중간 인증서는 `chain.pem`으로 지정을 하고 확인을 누르면 된다.

그러면 모든것이 끝나게 된 것이다. 그러나 한가지 마지막 스텝이 남아있다. Lets Encrypt는 90일 동안만 인증서가 유지가 되고, 30일이 지나야지 갱신을 할 수 있다. 그렇기 때문에, 매달 1번씩이라도 라이센스를 갱신하도록 만들어야 한다는 것이다.

그래서 우리는 이를 리눅스에 있는, 또는 시놀로지의 기능으로 제공되는 `cron` 또는 `스케줄링` 기능을 이용하여 이를 자동화 할 에정이다. 

![](/assets/img/post/2024-05-07-04.png)
![](/assets/img/post/2024-05-07-05.png)

## 6. 자동 갱신 설정

우선, 먼저 자동화를 하기 위해서 2가지의 선행이 필요하다. 

### 1. Synology의 인증서 저장 경로를 도커 경로와 매칭하기

그냥 아래의 순서를 따라 입력하면 된다.

```sh
$ sudo -i

$ WORK_DIRECTORY=/volume1/main/ssl # example
$ DOMAIN=example.com # example

$ DATA=`cat /usr/syno/etc/certificate/_archive/DEFAULT`
$ cd /usr/syno/etc/certificate/_archive/${DATA}
$ rm -rf * && \
    ln -s ${WORK_DIRECTORY}/etc/live/${DOMAIN}/cert.pem cert.pem && \
    ln -s ${WORK_DIRECTORY}/etc/live/${DOMAIN}/chain.pem chain.pem && \
    ln -s ${WORK_DIRECTORY}/etc/live/${DOMAIN}/fullchain.pem fullchain.pem && \
    ln -s ${WORK_DIRECTORY}/etc/live/${DOMAIN}/privkey.pem privkey.pem
```

그러면, 이제 `cerbot`을 이용한 자동 갱신을 쉘 코드를 작성하고, 실행만 한다면 이제 끝나게 됩니다. 여기서, 클라우드 플레어를 사용중이라면, `Cloudflare`의 API Token을 추가하여 자동화 시킬 수도 있습니다. 자세한 내용은 [[클라우드 플레어와 Certbot 연동]](https://certbot-dns-cloudflare.readthedocs.io/en/stable/#credentials)을 확인하시면 됩니다.

예시로는 아래의 코드와 같습니다.

`${WORK_DIRECTORY}/etc/cloudflare.ini` 을 생성하고, `dns_cloudflare_api_token = <API Token>` 내용을 기입하시면 됩니다.

```sh
WORK_DIRECTORY=/volume1/main/ssl # example
CONTAINER_NAME=certbot

docker run --rm \
  --pull=always \
  --name ${CONTAINER_NAME} \
  -e "TZ=Asia/Seoul" \
  -v "${WORK_DIRECTORY}/etc:/etc/letsencrypt" \
  -v "${WORK_DIRECTORY}/var:/var/lib/letsencrypt" \
  certbot/dns-cloudflare renew \
  --dns-cloudflare \
  --dns-cloudflare-credentials '/etc/letsencrypt/cloudflare.ini' \
  --dns-cloudflare-propagation-seconds 60
```

마지막으로, 30일 주기의 `Cron`은 시놀로지에서 `제어판` -> `서비스` -> `작업 스케줄러` 에서 등록하고, 위의 쉘 스크립트를 추가하시면 됩니다.

## 결론

정말, 힘든 과정을 거치게 되었고, 기존에는 맘 편하게 `DDNS`를 이용하거나 `Cloudflare`의 `Proxy` 서비스를 이용하여 무료 `SSL` 인증서를 기반으로 진행하였었습니다. 그러나 직접 `Let's Encrypt`를 발급 받아서 사용하니 이제 조금,,, 이상한 `DDNS`의 URL 주소를 사용하지 않고 자기 자신만의 도메인을 기반으로 사용할 수 있다는것이 조금 주소가 더 이뻐진것 같아 기분이 좋습니다. 

큰 의미없이 그냥 정말 단순히 기분이나 뽐내기용이다보니, 기분은 좋습니다. 그리고, `Cloudflare`의 `Proxy` 서비스는 특정 포트만 가능하다는 단점으로 인해서 `WebDAV` 서비스를 이용하기에는 꽤나 많은 어려움이 있었는데, 그런 것이 없어지니 조금 편한것 같기도 합니다.
