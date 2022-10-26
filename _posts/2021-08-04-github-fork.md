---
layout: post
title: iOS 어플 개발 하면서 기존 라이브러리에 대한 문제 해결한 방법
author: piorosen
tags: [github, git, library, fork, swift, spm, package]
hide_title: false
categories: [Blogging, Develop]
# feature-img: "assets/img/feature-img/"
# feature-img: "http://placehold.it/700x300"
# thumbnail: "assets/img/thumbnails/feature-img/"
# thumbnail: "http://placehold.it/200x200"
---

# 개요
아이폰 어플을 만들게 되면서 다양한 문제를 직면하게 되었고, 해결을 하기 위해서 여러가지의 방법을 시도하게 되었다. 여러가지 문제가 있었지만 그 중 Git과 라이브러리에 대해서 문제를 이야기 해보고자 한다.
첫번째로는 현재 사용 중인 패키지 매니저와 맞지 않아서 라이브러리를 수정한 내용이며 자세하게는 아이폰에서 여러개의 패키지 매니저가 있는데 SwiftPM을 고른 이유와 SwiftPM을 골라서 생긴 문제를 어떻게 해결을 하였는가. 이다. <br>

두번째로는 현재 사용 중인 라이브러리와 개발 중인 어플 간 디자인 불일치로 코드 수정이다. <br>


# Swift Package Manager (SwiftPM) 을 선택하게 된 계기

아이폰 어플을 개발 진행 하면서 최대한 최신의 내용으로 개발을 해보고자! 라는 목표를 가지고 있었다. <br>
그래서 Package Manager에서 여러가지를 고민 하게 되었고.
1. Carthage
2. CocoaPods
3. Swift Package Manager
중 어떤것을 이용하면 좋을까? 란 생각과 다른 팀원이 쉽고 처음 한다면 어떤것이 쉬울까? 라는 생각을 하게 되었다.

Carthage나 CocoaPods는 과거에서 부터 사용되었고 많이 사용하지만 라이브러리를 적용을 하거나 처음 빌드 할 때 명령어로 해야한다는 부분이 단점으로 생각이 되었다. 왜냐하면 처음 개발하는 사람에게는 이 명령어를 입력하고 종속성 관리라는 내용은 많이 어렵다고 생각이 들었다. (대학교 2, 3학년 기준) 그 외에 학교 수업을 진행 하면서 Visual Studio와 OpenGL과 같이 외부 라이브러리를 적용 해야할 때 주변에서 처음 환경 설정을 어떻게 하는지 많은 질문을 받다 보니 자연스럽게 Carthage나 CocoaPods는 난이도가 있고 처음 하는 사람에게는 거부감이 들 수 있겠다. 라는 생각을 하게 되었다.

그래서 비교적 난이도가 쉽고 명령어가 아니라 미리 빌드 환경 설정을 하고 다른 사람이 프로젝트 파일을 받으면 바로 빌드가 되는 SwiftPM에 매력에 빠지게 되었고, SwiftPM으로 결정하게 되었다. <br>
다르게 말하자면 iOS 만 개발 하기에도 처음이라서 어려운 사람이 많고, 패키지, 외부 라이브러리에 대한 개념 까지 교육 하기에는 시간이 부족해서 선택하게 되었다.

# SwiftPM 적용하고 생긴 문제점

SwiftPM은 생긴지 2년이 정도 지나서 지원을 하는 라이브러리가 많아졌고, SwiftPM만 지원하는 라이브러리가 있다. <br>
하지만 문제가 생기는 것은 메이저 라이브러리/프레임워크가 아닌 과거에 지원을 했던 라이브러리가 현재 지원이 중단이 된 경우 이다. 이러한 경우에는 SwiftPM은 지원을 안하고 Carthage나 CocoaPods만 지원을 한다. 이런 경우에는 2개 이상의 Package Manager를 사용하는 것도 하나의 방법이지만 그렇게 되면 처음에 SwiftPM을 선택하게 되는 이유가 사라지게 되므로 미지원하는것을 지원 하도록 수정을 해야 했었다.

대표적으로 진행을 했던 부분은 페이지 라이브러리가 필요 하게 되면서 라이브러리를 찾던 중 PageMenu가 눈에 들어왔지만 SwiftPM을 지원하지 않아 지원하도록 Fork 해서 수정하게 된 대표적인 예 이다.

[![이미지](https://raw.githubusercontent.com/uacaps/ResourceRepo/master/PageMenu/PageMenuHeader3.png)](https://github.com/Piorosen/PageMenu)

해당 라이브러리는 SwiftPM 지원하지 않고 CocoaPods와 Carthage만 지원을 하고 있다. 그래서 SwiftPM 이 지원 할 수 있도록 코드를 수정하고, Swift5.4 또한 지원 하도록 코드도 일부 수정을 하였다.

처음에는 기존 라이브러리를 SwiftPM으로 변환에 어려움을 겪었지만 실제로는 swift package init 을 입력하고, 파일의 위치만 옮겨주면 해결이 될 정도로 간단 하였다.

그래서 미지원 했던 라이브러리를 Fork를 한 뒤 개인 레포에서 작업을 해 사용을 할 수 있도록 만들었다.

# 문제가 있어 라이브러리를 직접 수정해서 해결한 경우.

어플을 개발 하면서 Target OS를 iOS 12로 잡게 되면서 생긴 문제 중 하나이다. 처음 개발 하면서 iOS 12에 대해서 지원 난이도는 어렵지 않고 비교적으로 쉬울 것이라 예상을 하였지만 그것은 절대로 아니였다.

iOS 12와 iOS 13에서 개발 도중 가장 크게 느꼈던 부분은 아래와 같다.
1. 다크 모드 지원
2. Entry Storyboard 지정
3. ContextMenu 지원
4. StackView의 배경 색상 지정

여기서 1번 2번, 4번은 쉽게 해결을 하였지만 3번의 ContextMenu는 아예 존재 하지 않아 라이브러리를 이용해서 구현을 해야 했었고, 적합한 라이브러리로는 아래와 같았다. <br>
그래서 아래의 라이브러리를 사용을 했다.

[![이미지](https://raw.githubusercontent.com/MarioIannotta/SwiftyContextMenu/main/Resources/logo.png)](https://github.com/Piorosen/SwiftyContextMenu)

그렇지만 애니메이션의 효과가 커지는 효과가 있어서 UI가 짤리는 현상이 있어 코드를 수정 해서 해결을 하였다.

# 정리

전에 개발을 할 때에는 최대한 나에게 맞는 라이브러리를 찾아서 적용을 할 생각을 했었지만, UI나 커뮤니티가 활성화가 되어있지 않는 언어, 프레임워크를 사용하는 할 때 개발에 어려움은 처음 겪어 보았다.
주로 C#이나 Python이나 PHP 언어와 같이 개발자가 많아 커뮤니티가 활성화 되어 있는 언어를 주로 했었지만 iOS나 Swift인 언어는 커뮤니티나 라이브러리가 많지 않았다.
그 덕분에 직접 라이브러리를 만들어야 하는 경우가 많았지만, 기존 라이브러리를 나의 입맛에 맞춰서 수정을 해서 적용을 우선시 하고, 개발한 내용이 버그나 수정을 한 내용이라면 PR(Pull Request)를 하는것도 좋을것 같다라는 생각이 들었다.

그리고 Fork에 대해서 왜 이것이 필요한가? 라는 생각을 많이 하게 되었는데 이번 기회에 PR이나, Fork에 대해서 왜 필요하고 어디에 쓸 수 있는지 알게 되었다.
