---
layout: post
title: 😎OpenGL하다 엔진⚙️ 만들었나❓  날 구해줘 팝콘쥬
author: piorosen
tags: [opengl, engine, graphics, fmod]
categories: [Blogging, Develop]
---

# 개요 
학교 수업 중 컴퓨터 그래픽스란 수업을 들으면서 OpenGL 2.0을 이용하여 벽돌깨기 게임을 구현하라는 과제가 나오게 되었습니다. 그래서 옛날에 오목 게임을 만들어 보면서 작지만 저만의 UI 라이브러리를 만든적이 있어 UI 라이브러리를 활용하면서 개발을 진행하면 편하겠다. 라는 생각으로 이미지를 통하여 깔끔하고, 물리엔진을 통하여 충돌을 해결 하자! 라는 생각으로 개발하게 되었습니다.

# 개발 관련 기간
> 1. 엔진 개발 : 2~3일
> 2. 물리 엔진 개발 : 1일
> 3. 이미지, UI 그리기 : 1일
> 4. 발표용 자료 만들기 : 1일
> 총 기간 : 3~4일

리소스 개발 해주신분 : [MFDO](https://github.com/oMFDOo)

# 적용된 기술

> 1. 깨소금 같지만 마우스 커서 커스텀
> 2. 마우스 입력 (클릭을 통한 반응)
>> hover, down, default 상태를 구현 함.
> 3. 이미지 출력기능을 이용한 애니메이션 구현함
> 4. DaC 와 같은 코드형 디자인
>> 버튼이나, UI를 객체를 생성하고, 관측영역에 대입을 해주면 자동적으로 렌더링함.
> 5. 맵 커스텀 기능
> 6. 숫자 폰트
> 7. FMOD를 통한 동시 재생 기능
> 8. Tick Rate를 통한 배속, 렌더링 틱 구현

# 내부 동작 원리

OpenGL에서 UI를 비교적 쉽게 구현하기 위하여 다른 프레임워크와 벤치마킹을 진행 하게 되었습니다. 그래서 과거 비교적으로 많이 사용했고, 로우하게 많이 사용해 보았던 Swift의 UIKit을 기반으로 진행 하였습니다. 다른 프레임워크를 비교한 것으로는 WPF, Winform, UIKit, QT가 있었습니다. 그 중 제일 로우 하고, 비교적 분석을 많이 했던것은 UIKit 이였고, Winform이나 다른것은 쬐끔 많이 달랐습니다.

그래서 UIKit과 유사하게 구현하고자 하였지만, 이 모든것을 구현한다면 3일 이라는 기간내에 완성하는것이 매우 어려울것이라 판단하여 선택과 집중을 하였습니다. 그래서 ViewController는 Scene이라는 개념이 되고, Scene안에 다양한 View가 존재하는 형식으로 진행 하였으며, View에 물리엔진을 넣기 위하여 View를 상속 받는 PhysicsView라는 추상 클래스를 만들어 물리 엔진이 필요한 요소는 해당 PhysicsView를 상속받아 구현하도록 하였습니다.

마우스와 키보드 입력이 발생한 경우 Scene이나 Application 단에서 적정한 View에 전파하는것이 아닌, 일단 모든 View에게 전파하고, View가 선택하여 자신에게 온 데이터 인지 아닌지 검증하여 실행 하도록 하였습니다. 그래서 검증 하는 코드 중 하나가 현재 View가 Hidden 인지 아닌지 체크 합니다.

즉 간단하게 그림으로 표현 한다면 아래의 이미지 처럼 됩니다.

![main](/assets/img/post/2022-06-05-render.png)

매번 렌더링이나, 키보드 이벤트가 발생한다면 Application에서 받고, 현재 활성화가 되어있는 Scene에게 전송하게 되고, Scene에서 모든 View에 전파합니다.
그래서 최적화와, 마우스 입력과, 이미지 출력 기능과 같은 베이스를 구현 하면 나머지 기능을 자유롭게 쓸 수 있게 됩니다.

Application에서 모든 기초 베이스를 담당하기 때문에 TickRate를 변경하거나, Tick을 발생하는것도 모두 Application에서 담당하므로, 일괄적인 변경, 특정 Scene에만 바꾸거나 하는 등 다양하게 할 수 있습니다.

# DaC, 코드형 디자인 이란?

기본적으로 내부 동작의 원리를 분석하게 되면 "Scene"에 연결이 되어 있다면 UI나, 무언가가 그려지게 됩니다. 그렇기 때문에 View 객체를 생성을 하기만 하더라도 그려지지 않습니다. 그리고 View간 관계형이나, View가 어떤 Scene에 있는지 어떤 요소가 있는지, 어떤 이미지를 쓰는지 코드로 선언을 해야합니다. 
아래의 이미지 처럼 View에 대한 관계성을 입력하고, 어떤 스프라이트를 그릴것인지 넘겨주어야 합니다.

![main](/assets/img/post/2022-06-05-dac.png)

그러면 자동적으로 Application의 주기에 따라, Scene이 렌더링이 될 때 그려집니다.



# 다운로드 링크
◆ 게임 다운로드 링크 : <Br>
https://github.com/Piorosen/opengl-popcornjoo/releases/download/v1.0.0/PopcornJoo.zip<Br>

◆ 게임 소스 코드 : <Br>
https://github.com/Piorosen/opengl-popcornjoo<Br>

# 발표 영상

[![main](/assets/img/post/2022-06-05-001.png)](https://youtu.be/E2IK1jFpJvI) 
발표 영상
{: .text-center } 이미지 

# 발표 이미지

![main](/assets/img/post/2022-06-05-001.png)
![main](/assets/img/post/2022-06-05-002.png)
![main](/assets/img/post/2022-06-05-003.png)
![main](/assets/img/post/2022-06-05-004.png)
![main](/assets/img/post/2022-06-05-005.png)
![main](/assets/img/post/2022-06-05-006.png)
![main](/assets/img/post/2022-06-05-007.png)
![main](/assets/img/post/2022-06-05-008.png)
![main](/assets/img/post/2022-06-05-009.png)
![main](/assets/img/post/2022-06-05-010.png)
![main](/assets/img/post/2022-06-05-012.png)
![main](/assets/img/post/2022-06-05-013.png)