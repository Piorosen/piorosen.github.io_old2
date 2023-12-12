---
layout: post
title: 온라인 SQL 저지 개발을 위한 Out of Docker 기반 개발지
author: piorosen
categories: [Blogging, Develop]
tags: [docker, go, out-of-docker, docker-in-docker, mariadb, network, container]
hide_title: false
---

# 개요

기존에 `C`, `C++`, `Python`언어와 같은 저지 사이트를 운영하면서 `SQL` 수업에서도 적용할 수 없을까? 라는 고민을 하게 되었고, 거진 2년동안 오픈 소스에 대해서 찾아보고 조사를 하였었다. 그 결과로는 완벽한 오픈소스는 없었고, 대부분 정상적으로 동작하지 않았었다. 그렇기에 개발 팀원을 꾸려, 단기간안에 SQL 저지 사이트를 구축하여 내년에 실적용을 목표로 개발하게 되었다. 

기존의 저지 사이트의 경우 입력과 프로그램 연산을 수행하여 결과 값에 대해서 `diff` 비교를 하면 되었었지만, SQL의 경우에는 동일한 값인지, 직접 개발하는 것이다 보니 `SQL Injection`에 대해서도 고민이 필요하였다. 그래서 이에 적합한 해답을 찾다 보니 시스템용 데이터베이스와 채점용 데이터베이스를 분리하여 실행하는 것이 옳다고 판단하게 되었다. 이를 통해 악의적인 SQL 쿼리를 요청하더라도, 시스템에는 영향받지 않는 시스템을 구성할 수 있을거라 판단하였다.

그래서 `Container`기반의 채점 시스템을 도입하게 되었고, 이를 제어하기 위한 `프로그램`에 대해 기술 명세와 자세한 스펙에 대해 정의가 필요했다. 명세 작성하는 과정에서 어떻게 구현할지에 대해 논점을 맞추면서 `Docker In Docker`와 `Docker Out of Docker`에 대해 논의가 진행이 되었었다. `Docker In Docker`은 `Docker`를 제어하기 위해 `Host`의 제어권을 가질 것인지, `Docker Out of Docker`은 `Docker`의 Agent 제어권을 가질 것인지에 대해 요구 분석 하였었다.

결론적으로는 `Host`의 제어권을 가지는 것보다 `Agent of Docker`의 제어권을 가지는 것이 더 보안적으로 안전할 것이라 판단하였고, 추가적인 개발할 내용이 많은 `Docker Out of Docker`를 선택하였다.

# 선택 과정

Docker는 Kernel(bootfs 나 rootfs를 통해서)을 공유한 시스템이다보니, 내부적으로 가상화를 하더라도 결과는 동일하다. 하지만, `Container`에서 한번 더 `bootfs`와 `rootfs`를 공유하는 가상화 시스템 구축하여 제공하는 방식인 `Docker In Docker`는 매우 보안상으로 위험하다. 이는 `Host`의 `bootfs`와 `rootfs`에 접근하고 제어하는 권한이 필요하기 때문이며, `Host`와 같은 권한을 가져서는 안되기 때문이다.

그렇기에 보안상을 위해서 `Container`를 선택했지만, `Container`로 인해서 보안상 문제가 생겨서는 안되므로, 간접적으로 `Docker Agent`를 제어하는 `Docker Out of Docker`를 선택하게 되었다.

`Docker Out of Docker`는 `Docker Agent`는 HTTP 웹 통신 기반으로 이뤄지며, 하나의 파일을 통해 데이터를 전달 할 수 있도록 제공되어지고 있다. 그렇기에 `Host`의 `docker.sock` 파일을 `Container`에 마운트만 시킨다면, 따로 권한 문제 없이 간접적으로 `Docker Agent`를 제어할 수 있다.

![](/assets/img/post/2023-12-13-01.png)


# 개발 중에 생긴 문제점

SQL 채점 부분의 프로그램을 개발 하면서 크게 2가지의 문제점에 봉착하였었다. 
- 첫번째로는 `Docker Out of Docker`을 통해서 생성된 새로운 `Container`와 통신을 할 것인가? 
- 두번쨰로는 생성된 새로운 `Container`를 제어 할 것인가? 이다. 

다행이도 `Container`의 네트워크에 대한 지식이 있었기 때문에 첫번쨰의 문제는 쉽게 해결이 가능했었고, 개념 증명을 거치면서 무난히 해결하였다. 두번째의 경우에는 `Container`의 이름을 제어가 필요한 경우 `Prefix`와 `Container` 생성시 출력되는 해쉬값을 통해 해결할 수 있었다.

첫번째 문제인 어떻게 통신할 것인가에 대한 문제는 `Docker Agent`를 통해서 하나의 가상 랜 드라이버를 생성하였고, `Container`와 `Container를 제어하기 위한 Container` 모두 같은 망으로 연결하면서 문제를 해결 할 수 있었다.

아래의 사진은 `Portainer`에서 연결된 네트워크와 관리하기 위한 `Prefix`가 붙은 컨테이너를 확인해 볼 수가 있다. 이를 통해 예시로 보여지는 `docker_test_for_judge`에 같은 망으로 묶여져 있다면 `mariadb_for_judge_HASH~`에 대한 네트워크로 접근 할 수가 있다. 또한, `Docker Agent`를 활용하여 `prefix`가 붙은 컨테이너에 대한 목록과 정보를 수집하여, `Container`의 `IP 주소`를 얻을수가 있으며, 얻은 IP 주소를 기반으로 통신을 수행할 수 있다.
(물론, `Container`에서 `Container`간의 통신이므로 같은 네트워크 상에 묶여 있어야 한다.)

Network|System
:---:|:---:
![](/assets/img/post/2023-12-13-02.png)|![](/assets/img/post/2023-12-13-03.png)
 
# 개발 관련 및 자료

현재 개발은 2023년 12월 13일 기준으로 진행 중이며, 추후 오픈소스로 개방할 예정이다. 또한, 기회가 된다면 소프트웨어 등록이나 특허, 논문을 작성을 할 계획이다. 개발은 현재 `Github`나 `Gitlab`이 아닌 직접 구축된 사내용 `Gitea`를 구축하여 개발을 진행하고 있다.

https://gitea.swdev.kr/

