---
layout: post
title: Nuget과 Vcpkg를 활용한 C++과 C# 패키지 관리
author: piorosen
tags: [nuget, vcpkg, cpp, c-sharp, dotnet]
hide_title: false
categories: [Blogging, Develop]
---

# 개요
교수님과 함께 연구과제를 진행하면서 하나의 Visual Studio 프로젝트 안에 C++ 프로젝트와 C# 프로젝트를 같이 개발을 진행 해야하는 문제가 발생하게 되었다. 가장 큰 문제는 온라인 기반인 Nuget이나 Vcpkg를 이용하여 개발을 진행 하게 된다면, 외부 망과 연결이 되어 있지않은 망인 경우 패키지 매니저를 온전히 쓸수가 없으며, 오프라인 모드가 가능한지 확인 해야 한다. Nuget은 오프라인일 때 라이브러리만 정상적으로 옮겨준다면 동작하지만, vcpkg는 거의 필수적으로 인터넷이 필요하기 때문에 vcpkg를 nuget으로 변환하고, nuget을 연결 하였습니다.

# vcpkg와 Nuget이 무엇인가?

과거 C++을 이용하여 개발을 할 때 매번 종속성을 관계를 구성하고, Makefile에 작성을 하고, 링크와 컴파일해야 했습니다. 하지만 프로젝트가 커지고, 방대해지고 오픈 소스란 개념이 발전하게 되면서 C++의 패키지 매니저를 Microsoft에서 새롭게 만들게 되었는데 그것이 vcpkg 입니다. vcpkg는 CMake와 Visual Studio 끼리 연동이 되기 때문에 Windows 환경이나 CMake 사용 할 수 있다면 MacOS, Linux에서 사용 할 수 있는 멀티 플랫폼 패키지 매니저 입니다.

Nuget은 오직 .Net 을 위하여 존재하는 패키지 매니저 입니다. 간혹 C++ 라이브러리를 Nuget 형식으로 만들고, 배포하는 경우가 존재 하기 때문에 C++에서 Nuget을 적용하는 사례가 존재 합니다. 비록 Nuget은 C#의 패키지 매니저이지만, C++에서 적용이 가능합니다.

가장 대표적으로 학교 수업에서 OpenGL을 이용한 수업이 있었는데, Nuget 패키지에서 NupenGL이라는 C++ 라이브러리를 활용하여 손쉽게 클릭만으로 개발 환경을 구축 할 수 있습니다.

# vcpkg에서 Nuget으로 변환이 가능한가?

![image1](/assets/img/post/2022-06-22-vcpkg-export.png)
VCPKG에서 Nuget 변환 모습
{: .text-center } 

와! 놀라워요! vcpkg에는 이미 nuget으로 변환하는것을 지원 하기 때문에 간편하게 바로 바꿀수 있습니다!

# 변환된 Nuget 어떻게 오프라인 모드로 쓰는가? 
![image1](/assets/img/post/2022-06-22-offline.png)
와! 따로 관리 되는 nuget
{: .text-center } 

Nuget은 어떻게 보면 효율적인 시스템 입니다. Nuget에서 패키지를 하나를 다운로드 하게 되면 프로젝트에 종속적으로 프로젝트에 관련 라이브러리가 깔리는것이 아닌, "시스템 전체의 패키지 소스"에 다운로드를 한 뒤 프로젝트에 관련 종속성을 복사하는 구조 입니다. 그렇기 때문에 nuget.org를 지우고, 로컬 URL 주소로 소스를 지정하게 되면 오프라인 모드로 사용할 수 있습니다.

# 그래서 잘 동작하든가에?

처음부터 오프라인 환경에 있고 반출 조차 허용 되지 않는 환경이였기 때문에 개발 환경인 Visual Studio 부터 오프라인으로 설치해야 했습니다. 그래서 꽤나 어떻게 해야하는지 골머리를 쌓았습니다. 그래서 Visual Studio를 오프라인 설치 하기 위해 검색을 하였고, 외장하드에 설치를 해서 가져갔습니다. [Visual Studio Offline Install](https://docs.microsoft.com/en-us/visualstudio/install/create-an-offline-installation-of-visual-studio?view=vs-2022) 

그리고 Visual Studio를 설치하고, Nuget 오프라인 설치 하기 위해 환경 구성을 하였습니다.  그랬더니? 정말 잘 동작하는것을 확인하였습니다! 와?!?!?! 잘 돌아가요!

두번째로 한 오프라인 환경 개발이였지만 정상적으로 동작하는것을 보고 신났습니다.
