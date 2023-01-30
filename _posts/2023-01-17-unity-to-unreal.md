---
layout: post
title: Unity에서 구매한 에셋을 Unreal Engine으로 변환
author: piorosen
categories: [Blogging, GameEngine]
tags: [unity, unreal, prefab, fbx-exporter, fbx, exporter]
hide_title: false
---

# 개요

프로젝트를 언리얼 엔진으로 진행 하던 중, 이쁜 3D 디자인이 필요해졌다. 그러나 이쁜 디자인이 유니티의 에셋스토어에 업로드가 되었다보니, 에셋스토어에서 구매한 제품을 언리얼 엔진으로 변환하는 과정이 필요해졌다. 그래서 이를 위해 짧은 여정을 표현하고자 작성하게 되었다.

# 과정

유니티 에셋을 언리얼으로 변환하는 과정은 간단하였다. 기본적으로 제공되는 내용은 완전한 변환은 불가능에 가깝다. 그 이유는 애초에 유니티는 C#으로 개발하는 언어이며, 언리얼엔진은 C++과 블루프린트로 이용하기 때문이다. 따라서 유니티와 언리얼간 자료 이동은 모델링 파일만 가능하다. 다만 유니티와 언리얼 모두 프리팹이나 액터로 구현된 3D 파일의 경우 하나의 모델링 파일로 추출은 가능하다.

우선적으로 에셋 스토어에서 구매한 내용을 FBX나 모델링 파일로 추출하기 위해서 유니티 프로젝트를 생성한 뒤, 에셋을 프로젝트에 임포트해야 한다. 그런 다음 아래의 패키지를 추가해야한다. 아래의 패키지는 공식으로 제공하는 추가 확장 프로그램이다. FBX Exporter 를 설치 한다.

[https://docs.unity3d.com/Packages/com.unity.formats.fbx@2.0/manual/index.html](https://docs.unity3d.com/Packages/com.unity.formats.fbx@2.0/manual/index.html)

![히힣](/assets/img/post/2023-01-17-fbx-exporter.png)

FBX Exporter를 설치 한 뒤 프리팹을 우클릭하여 추출 모두 하면 된다.

![히힣](/assets/img/post/2023-01-17-unity-rcport.png)

그 결과 아래처럼 모든 결과가 fbx로 추출 한 다음 언리얼엔진에서 임포트 하게 된다면 아래의 결과처럼 나오게 된다.

![히힣](/assets/img/post/2023-01-17-result.png)



