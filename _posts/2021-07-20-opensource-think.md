---
layout: post
title: 아이폰 어플 개발 하면서 느낀 오픈 소스의 장점, 단점과 고마웠던 점
author: piorosen
tags: [opensource, swift, kakao, library, slack]
hide_title: false
# feature-img: "assets/img/feature-img/"
categories: [Blogging, Develop]
# thumbnail: "assets/img/thumbnails/feature-img/"
---

# 개요
기존에 있던 안드로이드 어플을 아이폰으로 비슷하게 만드는, 포팅(Porting) 작업을 하고 있었다.
아이폰 어플을 옛날에 만들어 보긴 했었지만 SwiftUI 라는 애플에서 만든 새로운 UI Framework를 이용해서 개발을 하고 있었다.
하지만 포팅 작업을 할 때, 아이폰의 OS 점유율에 의해서 개발을 진행 해야 했기 때문에 최소 OS 버전을 12으로 잡고 개발을 하게 되었다.
SwiftUI 는 2019년 WWDC 에서 나왔으며, iOS 13 부터 지원을 하지만, StateObject이나, Window 기능이나, Image PickerView와 같은 내용은 결국엔 UIKit을 이용해서 만들어야 하기 때문에 iOS 12를 타겟으로 잡고 개발하게 되었다.

(iOS 13으로 올려도 마찬가지로.. UIKit으로 진행 했을 것 같다.)

<img src="/assets/img/post/2021-07-20-iphone-os.png" width="70%"/>
{: .text-center }

# 아이폰 어플 포팅 작업 하면서 생겼던 문제 들
가장 큰 문제는 안드로이드 어플로 작성이 되어 있던 내용 대부분이 안드로이드에 최적화 되어 있었기 때문에 안드로이드와 최대한 유사하게 만들어야 했었다.

예를 들어서 안드로이드만 존재하는 Toast 기능이였다. 

개발을 진행 할 때 안드로이드 어플 기능이 어떤 것이 있는지 로그인, 회원 정보 수정을 해보면서 결과를 Alert로 알림창이 나오것이 아닌, Toast로 나온것을 확인을 하여 Github에 Swift Toast라 검색하여 적용을 하였다.

그 외에 TextField 에서 아이폰과 안드로이드의 디자인이 달라서 라이브러리를 찾고, 스택오버플로우 에서 관련된 라이브러리를 찾고 그랬었다.
그리고 UI에서 폰트 크기나 padding 값을 주는 방식이 달라서 임의의 상수 값으로 넣기도 하였다.

예)
1. Android
 - dp
 - sp
 - pt, px

2. Iphone
 - pt

안드로이드의 dp 값을 아이폰의 pt를 변환이 온전히 되지 않아 조금 속상 했었다.

# iOS 개발하면서 라이브러리 탐색 문제

기존 어플과 최대한 유사하게 만들기 위해서 다양한 라이브러리를 찾고, 적용을 시켰다.
하지만 적용을 하는것은 쉽지만, 성능 비교를 하면서 원하는 라이브러리를 찾는건 매우 어려웠다.
그 중에서 찾기 쉬웠던 것도 있었지만, 명칭, 용어를 몰라서 라이브러리를 찾기 어려웠던 것도 있었다.

대표적인 예로 아래와 같다.

iOS 어플 개발 하면서 유명한 라이브러리를 찾아 적용을 시키는건 매우 쉬웠다.
예) 비교적 찾기 쉬운 라이브러리
 - [Kingfisher](https://github.com/onevcat/Kingfisher) : ImageView의 리소스 캐싱 및 관리
 - [Alamofire](https://github.com/Alamofire/Alamofire) : 네트워크 통신 관련 (URLSession의 래핑)
 - [Toast-Swift](https://github.com/scalessec/Toast-Swift) : 안드로이드와 유사한 Toast View
 - [Charts](https://github.com/danielgindi/Charts) : MPAndroidChart 4.0를 Swift로 옮긴 버전
 - [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON) : JSON 라이브러리
 - [SQLite.swift](https://github.com/stephencelis/SQLite.swift) : SQLite 래핑

비교적 찾기 어려웠던 라이브러리
 - [PanModal](https://github.com/slackhq/PanModal) : Bottom Sheet(하단에서 올라오는 화면)

비교적 찾기 쉬웠던 라이브러리는 조금만 검색해도 나왔었지만, 어려웠던 라이브러리는 완전히 정 반대였다.
용어와 명칭을 커뮤니티를 통해서 알고 있었지만, 원하던 라이브러리에서는 해당 용어를 사용하지 않아 문제가 발생 하였다.

* Bottom Sheet를 찾기 비교 했었던 라이브러리
 - [https://github.com/gaetanomatonti/BottomSheet](https://github.com/gaetanomatonti/BottomSheet)
 - [https://github.com/OfTheWolf/UBottomSheet](https://github.com/OfTheWolf/UBottomSheet)
 - [https://github.com/gordontucker/FittedSheets](https://github.com/gordontucker/FittedSheets)

위 3가지 모두 적용을 해보고 테스트를 해보았지만 현재 프로젝트에서 제일 적합 한건

[https://github.com/gaetanomatonti/BottomSheet](https://github.com/gaetanomatonti/BottomSheet)
위의 내용이였다.

그렇지만 NavigationView나 다른 라이브러리에 충돌이 날 것 같아 한번 더 찾아보기로 하였었다.

# 어떻게 PanModal을 찾게 되었는가?

과거 카카오톡 어플에서 사용한 오픈소스 목록을 보았던 적이 있었다.

카카오톡 오픈 소스 접근 방법 | 카카오톡 오픈 소스 리스트 | 카카오 뱅크 오픈 소스 리스트
:---: | :---: | :---:
![카카오톡 오픈 소스 접근 방법](/assets/img/post/2021-07-21-kakaotalk.png) |  ![카카오톡 오픈 소스 리스트](/assets/img/post/2021-07-21-kakaotalkOpensource.png) | ![카카오뱅크 오픈 소스 리스트](/assets/img/post/2021-07-21-kakaoBank.png)

위의 이미지 처럼 카카오 계열 어플 또는 소프트웨어는 "앱설정 / 오픈소스 라이선스" 안에 사용을 하였던 라이선스를 기재하고 있다.
그 덕분에 카카오 뱅크에 있는 Bottom Sheet가 어떤 라이브러리를 사용 했는지 알게 되면서 현재 포팅 작업 중에 있는 어플에 적용하게 되었다.

<img src="/assets/img/post/2021-07-21-panmodal.jpeg" width="40%"/>
{: .text-center }

카카오나, 네이버 경우에 MIT 라이센스를 이용 하더라도 따로 공간을 마련해 라이브러리와 어떤 라이선스에 영향을 받는지 표기하고있어 정말 좋았다.

# 후기

안드로이드 어플을 아이폰으로 옮기는 포팅 작업을 진행하고 있었는데, 엄청 많은 부분이 아이폰에서 안드로이드와 유사하게 제공 하는 라이브러리가 상당히 많다고 느끼게 되었다.
예를 들어서 MPAndroidChart를 그대로 포팅 작업을 통해서 지원하는 Charts 라이브러리가 대표적이였다.

그 외에 여러 Toast 라이브러리가 존재 했었고, Toast 라이브러리를 사용 하기 위해서 여러 오픈 소스를 비교 하였었다. 성능 비교 하면서 Toast-Swift가 제일 적임이고, 다양한 커스텀을 지원해서 채택하여 사용하고 있었다. 마침 네이버 메일에서 메일을 보내고 나면 아이폰임에도 불구하고 Toast로 메시지가 왔던걸 생각해 네이버 메일 오픈소스 사용한 목록을 찾아본 결과 놀랍게도 Toast(Objective-C) 를 이용하였다는 것 이였다.(같은 사람이 만들었고 언어만 다름) 

전에는 라이선스가 MIT가 아니면은 라이선스 문제로 인해서 거부감이 들었지만 (예로 GNU) 어떻게 보면 타 기업에서 어떠한 라이브러리를 통해서 서비스를 하고 있는지 빠르게 알 수 있는 기회가 될 거라 생각이 든다. 그리고 라이브러리를 찾을 때 깃허브에서 검색 또한 좋지만 한번 쯤 다른 업체에서 어떤 라이브러리를 사용해서 만들었는지 찾아보는것도 좋은 것 같다.