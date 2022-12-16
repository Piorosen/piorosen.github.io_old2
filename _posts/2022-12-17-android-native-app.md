---
layout: post
title: NDK를 이용한 독립 실행이 가능한 C++ 웹 서버 개발
author: piorosen
categories: [Blogging, Hobby]
tags: [arm64, arm, aarch64, cpu, ndk, cmake, gcc, android, adb, restbed, not-jni, not-root]
hide_title: false
---

# 개요
분명 이 글을 읽으러 온 사람은 JNI를 통해 C++ 언어를 호출할 것이라 생각할 것이다. 하지만, 이 글은 그 상식을 벗어난 순수 C/C++로 작성 한 뒤 독립적으로 실행이 가능한 실행 파일로 빌드하는 방법에 대해 기술한다. 즉 adb나 터미널 창을 열었을 때 실행이 가능한 프로그램을 만드는것이 목적이다. 그렇기에 JAVA나, Kotlin을 사용하지 않고 순수 C/C++로만 작성하여 Android 운영체제에 동작하는 방법을 설명한다.

# 기존 사례

어플리케이션에서 동작하는 프로그램을 개발하는 것이 아닌 실행(바이너리) 파일로 실행하는 사례가 존재할지 의문일 수 있다. 그러나 놀랍게도 존재한다. 그 프로젝트 이름은 [[Termux]](https://termux.dev/en/) 이다.  

![Termux](/assets/img/post/2022-12-17-termux.png){: width="616" height="557"}

해당 프로젝트는 node.js나, llvm, rust, 그 외 기본 GNU 계열의 프로그램을 사용 할 수 있도록 제공하고 있다. 이 방식이 가능한 이유는 앞서 설명했듯이 안드로이드 앱을 설치 후, 서버에서 미리 빌드된 바이너리를 다운로드 받아 "실행"하는 구조이다. 그렇기에 Termux와 같은 사례가 존재하므로 충분히 가능하다는 것을 알 수 있다.

# 사전 지식

먼저 우선 선행이 되어야하는 지식이 필요하다. 

1. 안드로이드는 특수한 리눅스 운영체제이다.
2. 안드로이드는 경량화된 운영체제이다.
3. 안드로이드 폰은 보안을 위해 커널 기능이 일부 "삭제"가 되어있다.
4. 반드시 기억해야한다. 안드로이드는 리눅스이며, 기능이 삭제가 된 것이다.
5. 그렇기에 JNI에서 C++을 사용할 수 있다는건, 빌드가 된다는 의미이다.

# 간단한 테스트 방법

자세한 방법은 아래의 방법 또한 있지만, [[깃허브 레포지토리]](https://github.com/Piorosen/Restbed-based-Android-Web-Server)로도 제공하고 있습니다. NDK를 이용해 크로스 컴파일 개발 환경을 구축하고, Restbed를 이용하여 C++ 웹서버를 구축합니다. 

## 어떻게 빌드 하나요? (For Android, In Linux)

```sh
$ git clone https://github.com/Piorosen/Restbed-based-Android-Web-Server
$ cd Restbed-based-Android-Web-Server
$ chmod +x bootstrap.sh
$ ./bootstrap.sh
$ make build
```

## 빌드한 결과물을 안드로이드에 넣는 방법  (Using ADB, In Linux)

```sh
$ adb devices | grep device
# default value: ./build/chacha, because project name is chacha.
# adb push ${executable file: chacha} /data/local/tmp
$ adb push ./build/chacha /data/local/tmp
$ adb shell chmod +x /data/local/tmp/chacha
$ adb shell /data/local/tmp/chacha
```

## 어떻게 테스트 하나요? (Using curl, In Linux)

```sh
$ curl --request POST 'http://{Android IP Address}:1984/resource' \
    --data-raw 'test: android C++ server.'
```

## 결과

Server Result|Build
:---:|:---:
![Server Communicate Test](https://raw.githubusercontent.com/Piorosen/Restbed-based-Android-Web-Server/main/document/result.png)|![build success message](https://raw.githubusercontent.com/Piorosen/Restbed-based-Android-Web-Server/main/document/build.png)



 
