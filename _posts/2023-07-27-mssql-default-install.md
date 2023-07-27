---
layout: post
title: MSSQL-Dev를 설치 후, 외부 도구 활용 
author: piorosen
categories: [Blogging, Develop]
tags: [mssql, heidisql, tcp/ip, ssms, ]
hide_title: false
---

# 개요

외부 업체에서 제공하는 데이터가 확장자명이 MDF, LDF 파일이였다. 해당 파일은, Microsoft의 SQL인 MSSQL에서 사용되는 확장자이다.

그래서, 해당 파일을 열기 위해 MSSQL을 설치하고 이를 외부(Python 또는 HeidiSQL)에서 접근 하는 방법을 정리하였다.

해당 과정에서, 발로란트라는 게임을 하기 위해서 가상화 기술을 사용하지 못하는 상황이다. 그래서 도커를 사용하지 않고, 직접 설치해야하는 번거로움이 있었다.

# 설치 과정

우선, 데이터를 Import를 수행하기 전에, Import 하기 위한 서버가 필요하다. 그래서 (도커로 설치하는게 젤 맘에 빠르고 편하지만) MS 사이트에서 MSSQL을 설치한다.

![](/assets/img/post/2023-07-27-01.png)

그런 다음, 컴퓨터를 1회 재부팅하여야 한다.

설치를 진행하게 되면 기본적으로, MSSQL의 Developer를 설치하게 될 경우, 본인 컴퓨터의 계정으로 생성이 되고, IP 정보 또한 본인 컴퓨터의 이름으로 할당이 된다.

그래서, MSSQL을 설치 한 다음 외부, HeidiSQL이나, pymssql을 이용하는 것이 불가능하다. 

![](/assets/img/post/2023-07-27-02.png)

따라서 위의 이미지 처럼, named pipe로 한 뒤, Windows 인증 사용을 수행하면 된다.

## 데이터 임포트

데이터를 추가하기 위해서, MDF와 LDF의 경우 HeidiSQL에서 추가하지 못한다. 이는 데이터를 파싱하는 프로그램이 따로 있다는 것을 의미한다. 그래서 SSMS(SQL Server Management Studio Management Studio)를 이용하여 데이터 추가하여야 한다.

![](/assets/img/post/2023-07-27-03.png)

로그인을 수행하는 과정은 동일하다.

![](/assets/img/post/2023-07-27-04.png)

로그인 이후, 좌측에 있는 `Obejct Explorer`에서 `Attach...` 를 누른다.

![](/assets/img/post/2023-07-27-05.png)

이후에 MDF 파일을 `Add` 하여 DB를 추가할 수 있다.

## 권한 계정 생성

TCP로 데이터 접근을 허용하기 위해서, 윈도우의 인증 시스템을 사용하지 않고, 로그인 할 수 있도록 제공해야한다. 따라서, 우선 새로운 로그인 정보를 만들기 위해서 `Object Explorer`에서 `Security`의 `Logins`를 우클릭하여 `New Login...`를 클릭한다.

![](/assets/img/post/2023-07-27-08.png)

그런 다음, 아래의 이미지와 같은 화면이 나타나게 되는데, 아이디, 비밀번호 순으로 지정한 다음 User Mapping에서 Allow 하고 싶은 DB를 선택하면 된다.

![](/assets/img/post/2023-07-27-09.png)

해당 작업만 마친다면 끝이나는게 아니다!, 기본적으로 윈도우 인증 시스템만 사용하도록 옵션이 되어 있으므로, TCP/IP로도 로그인이 가능하도록 열어주어야 한다.

## TCP/IP로 접근 허용

TCP/IP로 접근을 허용하기 위해서 `SQL Server (년도) 구성 관리자`를 실행한다. 

![](/assets/img/post/2023-07-27-06.png)

실행 한 다음, `SQL Server 네트워크 구성` 에서 `프로토콜 이름`에 있는 `TCP/IP`를 `사용 안함`에서 `사용`으로 변경하면 된다.

![](/assets/img/post/2023-07-27-07.png)

그러면, 모든 모든것이 끝나게 된다.

## 결과

`pymssql`을 이용한 결과, 아래의 이미지와 같이 정상적으로 커넥션에 성공하는 것을 볼 수 있다.

![](/assets/img/post/2023-07-27-10.png)

# 결론

발로란트만 아니였으면, 도커를 이용하여 쉽게 환경설정이 가능할텐데 아쉽다.
