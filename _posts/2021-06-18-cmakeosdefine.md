---
layout: post
title: CMAKE 빌드 시스템을 활용한 OS 판별 및 그 외
author: piorosen
tags: [cmake, make, c++, multiplatform, predefine, macro]
hide_title: false
feature-img: "assets/img/post/2021-06-18-cmake.svg"
categories: [Blogging, Develop]
# thumbnail: "assets/img/thumbnails/feature-img/"
---

# 개요

C++에 있는 pre-define 기능을 이용하여 OS와 컴파일러, 아키텍처를 파악하는 방법에 대해서 연구하고 조사를 해보았었다.
여기서 C++의 빌드 시스템 중 하나인 CMAKE를 이용하여 OS를 판단 하고, System 명령어를 통해서 아키텍처를 확인 하는 방법에 대해서 정리를 해 보았다.

# CMAKE 란?

CMAKE는 간단하게 설명을 하자면 gcc나 msvc와 같은 컴파일러에게 so나 lib와 같은 파일로 먼저 빌드를 한 뒤 빌드가 된 파일을 모두 하나로 묶는 링크 작업을 하여 실행 파일을 만든다. 하지만 이러한 과정이 반복적이고, 노가다성이 높으므로 이러한 문제를 해결 하기 위해서 Makefile 이라는 빌드 시스템이 나오게 되었다. 그렇지만 Makefile 이라는 것 자체가 헤더 파일과 소스 파일 연관 관계, 링킹을 할 때 라이브러리 종속성과 같은 모든 내용을 기입을 해야하는 하므로 코드를 일부를 변경하게 된다면 모든 Makefile을 수정 해야하는 문제가 있으며, 자칫 잘못 했다가는 Git에서 충돌이 날 가능성이 높다.

그래서 Makefile을 대신 만들어주는 도구가 필요하게 되었으며 Makefile을 만들어 주는 도구인 CMAKE가 나오게 되었다. CMAKE는 소스 파일을 읽어 자동적으로 헤더파일이나 연관 관계를 정의하고 Makefile을 만들어 준다.

# CMAKE 이외의 빌드 시스템

빌드 시스템이라고 말을 하게 되면 상당히 어려운 개념이라고 생각하게 된다. 하지만 정말 우리 근처에서 많이 접하고 있으며, 이름만 생소하다는걸 알 수 있기도 하다.
예를 들어서 Android에서 외부 라이브러리 연결(Dependency), Target Version(OS 버전) 등 다양한 내용을 정의 할 수 있던 Gradle, Ninja, Maven, Ant, C# 문법으로 빌드 시스템 구현 하는 NUKE 등 정말 다양한 빌드 시스템이 있다. 

즉 빌드 시스템이라 말하는 것은 소스 코드와 외부 라이브러리를 묶어서 하나의 파일, 즉 실행 파일이나 라이브러리로 만들어 주는 시스템 이다.

# 코드

CMAKE에서는 기본적으로 OS 판별하는 기능과 System 명령어를 통해서 OS 버전이나 현재 상태에 대하여 값을 가져 올 수 있다.
그리고 [기본적으로 지원을 해주는 미리 정의가 된 변수](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html#id4)가 있다.

```cmake
# Add Dependecy Library Directory
if(APPLE)
	MESSAGE(STATUS "OS : APPLE")
	set(OS macos)
elseif(WIN32)
	MESSAGE(STATUS "OS : WINDOWS")
	set(OS windows)
elseif(UNIX)
	MESSAGE(STATUS "OS : LINUX")
	set(OS linux)
else()
	MESSAGE(FATAL_ERROR "your system is not supported")
endif()

# OS 마다 다르게 Source 디렉토리 지정 함
set(DIR_GRAPHICS lib/${OS}/Graphics)
```

운영 체제를 판단 한 뒤에 코드 상에서 쓸 수 있는 pre-define 코드 또한 작성이 가능하다

```cmake
# '#define OS WINDOWS' 와 같음
add_definitions( -DOS=WINDOWS )

if (APPLE)
    add_definitions( -DOS=APPLE )
endif()
```

이 처럼도 사용이 가능하다.

# 마무리

CMAKE 를 이용해서 정말 다양하게 멀티 플랫폼 지원이 가능하고, 빌드 조건도 걸 수 있다는걸 알게 되었다. 예를 들어서 특정 운영 체제, 컴파일러, IDE를 제한 할 수 있다는 점이 있다는 것 이였다. CMAKE 기능을 단순 하게 빌드를 해주는 기능이라 생각을 하였는데. 이번에 포스팅을 하게 되면서 CMAKE는 단순 빌드 하여 결과를 나타내는 것이 아닌 빌드 하기 전 모든 행위가 가 될 수 있다고 생각을 하게 되었다. 그렇다면 CMAKE를 목적은 Continuous Integration 이지 않았을까? 라는 생각을 하게 되었다.