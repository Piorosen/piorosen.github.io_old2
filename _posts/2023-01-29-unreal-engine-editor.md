---
layout: post
title: Unreal Engine의 에디터에서 반복성을 최소화하기 위한 툴 구현
author: piorosen
categories: [Blogging, UnrealEngine]
tags: [unreal, engine, macro, editor, blueprint]
hide_title: false
---

# 개요
정말 신기하게도 이런 신기한 작업을 하게 될지 몰랐다. 우선적으로 하게된 업무는 3D 폴리곤을 Approximate Convex Decomposition(ACD)를 수행 한 뒤, 이를 언리얼 엔진에 적용하는 것이다. 말이 어렵지만 간단하게 3D 폴리곤을 사각형이나 기본 도형만을 이용하여 구성되게 변환하는 알고리즘을 적용한 뒤, 결과를 언리얼 엔진에 적용해야하는 업무이다. 하지만 심각한 문제로는 하나의 3D 폴리곤에서 100 ~ 200 여개가 넘는 폴리곤이 나오게 되면서 이를 반복적으로 수행하는 것이 필요해졌다. 그래서 반복성을 최소화하기 위해 매크로가 필요함에 따라 언리얼 에디터에서 확장 기능을 이용할 수 있다는 것을 알게 되었다.

# 정리

언리얼 엔진에서 기본적인 커스텀은 지원하지 않으므로, 공식적으로 지원하는 에디터 플러그인을 이용하여 설치하여야 한다. 자세한 설명은 아래의 스크립트를 자동화하는 방법이 나와있다.

https://docs.unrealengine.com/4.27/ko/ProductionPipelines/ScriptingAndAutomation/Blueprints/

만드는 방법은 총 2가지의 방법과 2가지의 언어(Blueprint, Python)로 작성을 할 수 있으나, 필자는 블루프린트로만 사용 하였다. 방법으로는 Unreal Engine의 Actor와 Editor에서 상단 메뉴에서 실행하는 2가지 방법이 존재한다.

우선 액터에서 생성하는 방법은 아래와 같이, 액터를 생성 할 때 에디터 액터 상속 받아 구현해야, 기능이 나타난다.

상속을 받은 뒤에 고냥 코드를 짜면 끝이난다. 필자의 경우에는 액터에 추가한 하위 컴포넌트 중 Static Mesh Component라면 모든 Static Mesh에 컬리전 박스를 생성하는 코드를 작성하였다. 모든 코드를 작성 한 뒤에 결과를 확인한다면 아래 처럼 결과를 확인할 수 있다.

![a](/assets/img/post/2023-01-29-Editor.png)
![a](/assets/img/post/2023-01-29-UE.png)

![a](/assets/img/post/2023-01-29-Result.png)
