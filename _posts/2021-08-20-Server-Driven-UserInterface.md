---
layout: post
title: Server Driven UI. 서버가 주도하는 사용자 경험
author: piorosen
categories: [Blogging, Develop]
tags: [server-driven, ui, ux, dotnet, reflection, swift, swiftui, json]
hide_title: false
---

# 개요
UI를 개발하면서 정말 궁금하고 꼭 해결하고 싶었던 난제가 하나 있었다. 어떻게 해야지 어플을 재 배포 하지 않고 UI를 변경을 변경이 가능 할까? 라는 내용이였다. 다양한 방법으로 UI를 동적으로 변경하는 기술을 찾아보기도 하였지만 확실한 답을 내리지 못했었다. 하지만 최근에 iOS 세션을 찾아 보다 SwiftUI와 Components 패턴을 합쳐서 서버 주도형 유저 인터페이스(Server Driven User Interface)를 읽게 되면서 이러한 방식으로 UI를 변경이 가능하겠구나! 라는 생각을 하게 되면서 기억에 남기고, 나중에 다시 개발을 할 때 찾아보기 위해서 기록으로 남기고자 한다.

# 왜 서버 주도해서 UI를 바꾸고자 하는가?

iOS 어플이나, C#을 하면서 UI를 디자이너 또는 기획의 변경으로 인해서 디자인이 바뀌게 되는 경우가 많았다. 개발 중인 상태라면 새로운 브랜치를 열어서 작업을 하면 되지만, 런칭을 한 스마트폰의 어플리케이션 경우에는 사용자가 버전 업데이트를 진행 해야하는 문제가 있습니다. 물론 웹 페이지로 한다고 하면 사용자가 업데이트를 할 필요가 없이, Front-End 서버에 배포만 하면 되긴 합니다.

> ### [참고1. 서버 주도 UI가 필요한 이유](https://medium.com/@kimdohun0104/server-driven-ui-feat-flutter-87fcbb04e610)
> 1. UI를 더 추가하기 위해서 화면과 코드를 새롭게 작성해야 합니다.
> 2. 사소한 UI 변경에도 코드를 다시 작성하여 배포해야 합니다.
> 3. 스토어에 따라 배포가 바로 적용되지 않고, 검증 기간이 필요할 수 있습니다. (지연 시간 발생)
> 4. 사용자가 앱을 업데이트하지 않는 경우엔 변경 사항을 적용할 수 없습니다.

이 처럼 사용자의 결정으로 업데이트가 된다고 하면, 사용자가 업데이트를 하지 않는다면 배포를 했던 모든 버전을 지원 해야 한다는 의미가 된다.(지원 하기 위해서 시스템 리소스를 계속 사용 해야 한다는 것이다.)
그렇지만 Server Driven UI(이하. SDUI)를 이용하게 된다면 UI가 서버에 의해서 결정이 되고, 이벤트 처리도 서버에서 어떻게 할 지 정의를 한다는 의미 입니다. SDUI란 기술을 활용을 한다면 UI 서버를 따로 구축이 가능 할 것이고, 회사 측에서 별도로 SDUI를 위한 UI Designer를 만들어서 적용 하면 자동으로 배포되는 시스템 또한 만들 수 있습니다.

# SDUI 개념과 동작 원리

들어가기 전 한 문장으로 요약을 하자면 서버에서 UI 구성할 데이터를 받고, 이 데이터로 어떻게든 사용자 화면에게 보여주기만 하면 된다.

SDUI의 개념은 정말로 단순하고 구현 방법은 많이 있다. 그렇지만 가장 많이 적용을 하고 사용하는 대표적인 예시를 들어본다. SDUI는 말 그대로 서버 주도 UI 이다. 그리고 현재 다니고 있는 동의대학교에서 출석 앱이 대표적으로 SDUI 기술을 채택해서 사용 하고 있다. 과거 동의대학교 출석 앱의 출석을 안해도 출석이 되는 취약점을 찾으면서 알게 되었다. 출석 앱에서는 동작원리는 아래와 같다.

SPA(Single Page Application)와 Angular.js를 이용해서 만들어져 있었고, 동작원리는 아래와 같다.
1. index 에서 먼저 최초 화면, Splash Screen 나타 내면서 리소스를 다운 받음
2. UI Components(버튼, 이미지뷰, 텍스트뷰)를 먼저 다운로드를 함.
3. index 파일에 정의가 되어있는 스켈레톤 스크린을 출력 함.
4. UI를 렌더링 하기 위해서 GraphQL로 서버에 필요한 데이터를 요청 함.
5. GraphQL로 받은 데이터를 파싱하여 UI Components로 화면을 구성 함.

글로 표현 하면 어렵지만 조금 이해하기 쉽게 이미지로 표현을 한다면 아래와 같다.
![](https://camo.githubusercontent.com/3777557253513dac5d1ce0711f043de56d0f90b7dbd294ce549c194e72bded6d/68747470733a2f2f6d69726f2e6d656469756d2e636f6d2f6d61782f313430302f312a65306361714f4a616e51646c377976725531593070672e706e67)
[참조 이미지](https://github.com/AnupAmmanavar/SwiftUI-Server-Driver-UI)

# C# 으로 만들어 보았던 유사 SDUI 라이브러리

C#으로 만들어 보았던 ["SDUI Library"](https://github.com/Piorosen/Json-Custom-Designer) 라이브러리 이다. 만든지는 2019년 2월에 개발을 완료 했다. 만들게 된 계기는 친구가 음악 재생 프로그램을 요청 했었다. 이때 프로그램 기능 정의가 사용자가 프로그램의 디자인, 크기, 기능 까지 모든걸 수정이 가능해야 한다. 라고 기능 정의를 내리게 되면서 그에 맞는 라이브러리를 개발 해야 했었다. 서버 주도로 UI가 바뀌는 부분은 아니지만 사용자 주도로 UI가 바뀌도록 해야 하므로 유사하다고 볼 수 있다. 그리고 데이터를 전달 받는 주체가 사용자에서 서버로 바뀐다 하면 충분히 SDUI로 전환이 될 수 있는 라이브러리 이다. 

C# 으로 만들었던 라이브러리는 내부 동작은 Json Parser와 Reflection 기능을 이용해서 만들었다. Reflection을 다른 이름으로 부르자면 C++에서는 RealTime Type Information(이하 RTTI) 이라고 불린다.

닷넷에서 제공해주는 리플렉션은 내부 동작이 정말 아름답고 사람을 황당하게 만들지만(이야기하자면 IL 코드를 런타임 중에 만들어서 동적빌드를 사용하는건 사람을 와! 라고 외치게 됩니다.) 한번 이해하면 용도는 정말 다양합니다.

아래는 미리 정의했던 프로토콜에 대해서 기록을 합니다.

<center>
<iframe width="800" height="400" src="https://serviceapi.nmv.naver.com/flash/convertIframeTag.nhn?vid=6A3EDEA3DB5E193085FCF5FF646C808E7444&outKey=V1270fff8806136fa4fdf0dfa6debdbca80b1bf507a0c39f2faab0dfa6debdbca80b1" frameborder="no" scrolling="no" title="NaverVideo" allow="autoplay; gyroscope; accelerometer; encrypted-media" allowfullscreen></iframe>
<p>리플렉션을 활용한 예시</p>
</center>

이 처럼 데이터를 파싱해서 결과물로 출력이 가능한 형태라면 데이터를 받아들이는 방식이 네트워크 이거나, 시리얼 통신으로 들어와도 대응이 가능하다고 생각이 듭니다. 

# iOS나 AOS에서 SDUI를 적용 한다면?

닷넷의 Winform은 UI배치를 절대값으로 하므로, UI를 임의 배치가 가능하지만, WPF(Windows Presentation Foundation)나, Veu.js, Android, UIKit(iOS) 경우에는 Anchor나, UI간의 비율로 형태를 지정하는 경우가 많다. 그래서 UI의 공통적으로 사용하는 레이아웃을 먼저 구현을 하고 해당 레이아웃에 배치가 가능한 UI 컴포넌트를 미리 만들어서 사용 할 수 있도록 따로 구현을 해야할 것이다.

[SwiftUI 에서 SDUI를 구현 한다면?](https://github.com/AnupAmmanavar/SwiftUI-Server-Driver-UI) 여기서 구현체를 본다면 먼저 interface인 UIComponent를 먼저 정의를 하고 UI Component 에서 사용하는 데이터를 UI Model로 정의 하게 되어있다. 두번째로는 서버에서 UI 정보를 받아서 UI Component와 UI Model로 분리 해서 UI를 만드는 작업을 한다. 이러한 형식으로 구현을 한다면 미리 UI Component를 만들어서 배포를 하고, 그에 맞춰서 UI를 수정이나 디자인을 변경 하도록 하면 좋을것 같다. 

추가적으로는 UI Component를 새롭게 만들어야 하는 경우나 애니메이션을 새롭게 구현 해야한다면 상당한 문제가 발생 하겠지만, 그러한 예외가 최대한 발생하지 않도록 하는것이 좋을것 같다. 그렇게 된다면 UI 부분이 네이티브로 구현을 해야한다면 최고 이긴 하겠지만 Vue나 Xamarin, React와 같은 멀티플랫폼을 지원하는 프레임워크 쪽에서 SDUI 라이브러리가 활발하게 활성화가 되어있지 않을까 라는 생각을 해본다.

# 마무리

Server-Driven UI 에 대해서 공부를 해보았는데 이미 알고 있었던 개념이 많았고, 놀라운건 이미 C#이라는 언어로 구현을 해보았다는 것이 많이 신기했다. Componets 패턴은 독립적인 새로운 기능을 추가하기 위한 기능이라는 건 알고 있었지만 이렇게 UI 쪽을 컴포넌트로 만들고 서버에서 UI를 정의하는 건 많이 신기 했다. 서버 주도 디자인은 컴포넌트가 아니라 다양한 형식으로 구현이 될 수 있을수 있으며 분명히 생각이 드는 부분은 상업적으로 SDUI를 쉽게 만들어서 판매 또는 배포를 하고 있을것 같다. 

대표적으로 SDUI나, UI Component나 컴포넌트 패턴이 활발하게 사용이 되는 분야는 게임 쪽이라 들었다. 왜냐하면 앱스토어나 구글 플레이 스토어에서 업데이트를 진행 하는것 보다. 인 게임에서 업데이트를 진행 하는 것이 리소스 관리나 소프트웨어를 강제 할 수 있을 것이며, 매번 스토어에서 심사를 받는 과정으로 인해서 스트레스를 받지 않아도 되며, 이벤트를 진행 할 때 바로 즉각적으로 대응을 할 수 있기 때문이다. 확실히 게임쪽은 Metal이나, Vulkan과 같이 특정 위치에 렌더링이나 이벤트 처리를 구현을 직접 해야 하는 부분이 있다 보니 그래픽 부분은 따로 처리를 하고 Graphic의 윗 단의, 통합 레이어 쪽에서 C++ 형식으로 개발을 한다면 중복 개발이 발생하지 않을 것이며, 사용하기 많이 편할것 같다.

[마치 이러한 GAL 처럼](https://engineering.linecorp.com/ko/blog/line-ar-rendering-engine-yuki-elsa/)