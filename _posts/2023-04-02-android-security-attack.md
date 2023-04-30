---
layout: post
title: 안드로이드/JVM의 중간 언어(Byte Code)로 인한 취약점을 이용한 공격
author: piorosen
categories: [Blogging, Security]
tags: [decompile, security, attack, encrypt, jdx]
hide_title: false
---

# 개요

JAVA는 C/C++ 계열의 컴파일 언어와 다르게, 중간 언어(Java의 경우 Byte Code)로 컴파일이 된 후, JVM(Java Virtual Machine)에서 실시간으로 기계어로 실행이 되는 구조이다. 특히 JVM에서 실행 되기전 언어인 Byte Code는 기계어로 변환이 되기 전 언어 이므로, Byte Code를 Java나 Kotlin 언어로 변환이 가능하다. 어떻게 본다면, LLVM의 IR 언어와 같이, IR 언어(중간 언어)로 변환하기 위해 C/C++ 언어와 같이 프론트엔드가 있는 형태이다.

그렇다 보니 JVM에서 구동되는 Byte Code 또한 class가 존재하고, 객체 지향 언어이다. 즉, Java 언어를 컴파일 된다면 class 또는 jar 파일 형태로 나타나게 된다. 자, 그렇다면 어플리케이션 또한 JAVA/Kotlin으로 구현이되어 있다면 어딘가에 Byte Code가 존재할 것이고, 이를 jar 또는 class로 변환이 가능하다는 뜻이다.
이를 악용하여 어플리케이션 단에 보안 코드가 있는 경우 apk를 디컴파일하여 소스코드 분석이 가능하다.

하지만, JAVA/Kotlin 코드를 Byte Code로 변환하는 과정에서 코드 최적화하는 과정을 거친다면, 원본 코드로 복구하는게 불가능 할 수 있다. 그렇기에 디컴파일러 툴과, 소스코드 뷰어를 통해 원본 코드를 완벽하게 변환이 불가능하므로, 공격이 불가능 할 수 있다. 이런 경우 공격하는 방법과 아이디어 전환이 필요하다. 어떻게 하면 원본 코드를 볼 순 없지만, 볼 수 있는것과 같이 공격하기 위해 어떻게 해야하는가?

# 공격

시나리오를 작성해보자, 우리는 Byte Code를 가지고 있다. Byte Code만 있다면 소스코드 실행이 가능하다. 또한 객체 지향 언어의 특징을 가지는 JAVA/Kotlin으로 인하여 파일마다, class란 파일이 생성이 된다. 그렇기에, 특정 class 파일만 교체가 가능하다.

즉 정리하자면 아래와 같은 공격을 할 수있다.

1. 이미 빌드된, class 파일을 교체하여 악의적인 코드를 삽입하여 공격한다.
2. apk를 jar로 변환한 뒤, 자바 프로젝트에 추가하여 라이브러리 형식으로 공격한다.
3. IntelliJ에서 디컴파일러를 통해 디컴파일 된 결과를 추출하여 빌드한다.

필자의 경우 2번 시나리오로 공격하여 진행하였다. 이유는 일부에서, 난독화로 인하여 코드를 읽을 수 없거나, 디컴파일링을 완벽하게 수행하지 못하는 경우가 존재한다. 따라서, 2번 공격으로 우선적으로 실행하여 테스트를 진행하였다.

# 시나리오

공격하는 시나리오는 아래와 같다.

1. APK 파일을 JAR 형식으로 변환한다. 변환하는 방식은 [[dex2jar]](https://github.com/pxb1988/dex2jar) 를 통해 jar로 변환하였다.
2. IntelliJ에서 [Jar Library Import](https://atoz-develop.tistory.com/entry/JAVA-IntelliJ-IDEA-jar-%ED%8C%8C%EC%9D%BC-export-import-%EB%B0%A9%EB%B2%95)를 수행한다. 
3. 라이브러리 형식으로 임포트가 되었으므로, 특정 회사가 구현한 모든 기능이 사용가능하다.
4. 만약 이때, 암호화 키가 복호화가 불가능한 경우, En/Decrypt 함수를 통해 암호화/복호화가 가능하다.

# 검증

자세한 방식은 설명하진 않겠지만, dex2jar를 통하여 apk를 jar로 변환하고, jar를 라이브러리 형태로 임포트하였다. 그 결과 정상적으로 라이브러리로 인식이되었으며, apk에서 사용하던 기능을, 그대로 사용이 가능해졌다.

실험 검증을 하기 위해 암호화 함수 호출하여 서버에서 정상적으로 인식하는지로 확인하였다. 추가적인 실험을 한 결과, 정상적으로 동작하는것을 확인하였다.

![a](/assets/img/post/2023-04-02-01.png)
![a](/assets/img/post/2023-04-02-02.png)
![a](/assets/img/post/2023-04-02-03.png)
