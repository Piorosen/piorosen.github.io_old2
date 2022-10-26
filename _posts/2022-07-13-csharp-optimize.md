---
layout: post
title: Reflection을 활용한 관리자 페이지 구현
author: piorosen
tags: [reflection, dotnet, admin, option, ]
hide_title: false
categories: [Blogging, Develop]
---

# 개요
무언가 개발하게 되면 사용자를 위해 다양한 옵션이 존재 할 수 있습니다. 대표적으로 예를 들어 네트워크 통신 서버의 주소를 설정등 다양한 테스트를 위한 옵션이 존재 할 수 있습니다. 해당 값을 바꾸기 위해 따로 환경을 만들고, 코드를 작성하게 된다면 코드의 비효율성과 반복 노가다성이 존재 할 수 있으므로 미리 제작된 라이브러리 필요성이 있습니다.

# [리플렉션은](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/concepts/reflection) 무엇인가?
리플렉션은 어셈블리, 모듈 및 형식을 설명하는 개체를 제공해 줍니다. 즉 실행 도중에 타입과, 값, 이름을 알 수있습니다. C#에서 리플렉션은 객체가 어떤 타입이며, 어떤 이름인지, 어떤 네임스페이스에 있는지 등 모든 정보를 가져올 수 있습니다. 추가적으로 C#은 IL언어라는 중간언어를 통하여 실행이 되며 하이브리드 언어이기 때문에 인터프린터를 통하여 실시간 코드 수정 및 실행이 가능합니다. 그래서 런타임 중에 OpCode를 추가하여 클래스를 만들고, 함수를 지운다거나 함수를 만들 수 있습니다. 

대표적인 예로 [Harmony](https://github.com/pardeike/Harmony) 라이브러리가 존재 합니다. 해당 라이브러리는 딕셔너리 형태로 함수를 만들고, 생성이 가능하며, OpCode를 직접 호출하여 추가 할 수도 있습니다.

특히 C++은 컴파일 언어이기 때문에 C++만으론 리플렉션을 이용하는것은 불가능합니다. 그래서 RTTI 기능이 있고, 해당 기능은 컴파일 할 때 추가적인 코드나 심볼이 추가가 되는 형태 입니다. RTTI 기능이 있으면 당연히 속도가 느려지게 됩니다. 특히 dynamic_cast<T> 라는 캐스팅 연산자는 RTTI 중 하나 이기 때문에 RTTI 기능을 끈 상태라면 오류가 납니다.

# 어떻게 동작하는가?

정적이나 특정 변수를 관측하게 한 뒤 리플렉션을 활용하여 해당 객체에 존재하는 프로퍼티나, 변수 목록을 가져와 해당 변수를 제어 할 수 있습니다. 객체에 있는 변수를 자유롭게 접근하고, 제어가 가능하기 때문에 public static으로 정의가 된 A,B,C_Option을 읽어오고, 쓰기를 진행할 수 있습니다.

```cs
// 시각화 하고 싶은 옵션 목록들
public static class Option {
    // 다양한 옵션
    public static long A_Option = 20;
    public static long B_Option = 30;
    public static long C_Option = 30;
}
```

위와 같은 옵션이 있다고 한다면 아래의 이미지 처럼 시각화 할 수 있고, 현재 값을 읽어올수 있습니다.

|설정값과 변경되는 모습|결과의 모습은 대단해요!|
|:---:|:---:|
|![main](/assets/img/post/2022-07-13-op1.png)|![main](/assets/img/post/2022-07-13-op2.png)|
|![main](/assets/img/post/2022-07-13-op3.png)|![main](/assets/img/post/2022-07-13-op4.png)


![main](/assets/img/post/2022-07-13-op5.png)
와! 실제 구현 코드!
{: .text-center } 

# 다른 방법은 또 어떤게 있나요?

리플렉션은 정적이 아닌, 동적타임 때 변수의 이름이나, 설정된 네임스페이스, 특정 변수 이름으로 값을 읽거나 쓰기가 가능합니다! 그래서 특정 시간마다 로그 값을 출력하거나, 데이터 수집, 특정 데이터에 쓰기, 원하는 객체에 값을 적용하기 등 다양한 분야에 쓸 수 있다. 특히 Json의 데이터를 입력하면 GUI를 커스텀 할 수 있는 기능도 구현할 수 있습니다.

[[Json을 넣게 되면 디자인을 자유롭게 바꿀수 있어요!]](https://github.com/Piorosen/Json-Custom-Designer)

그리고 특정 시간마다 특정 변수를 관측하면서 로그를 출력 할수 있어요!
