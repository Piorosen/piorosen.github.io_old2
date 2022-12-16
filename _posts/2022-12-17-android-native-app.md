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