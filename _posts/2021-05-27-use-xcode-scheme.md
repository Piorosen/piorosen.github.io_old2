---
layout: post
title: IDE에 있는 Scheme 를 이용하여 효율적인 작업 분담 및 개발 내용
author: piorosen
tags: [xcode, mac, scheme, git]
hide_title: false
categories: [Blogging, Develop]
# feature-img: "assets/img/feature-img/"
feature-img: "assets/img/post/2021-05-27-xcode.png"
# thumbnail: "assets/img/thumbnails/feature-img/"
---

# 개요
연구과제를 통해 기존 Android 어플을 iOS로 포팅 작업 하고 있고, 개발 인력은 총 4명, 각자, Bluetooth, Network, Graphic User Interface(이하 GUI) 를 나눠서 개발을 진행 하고 있었다.

처음 프로젝트 계획을 세웠을 때 Bluetooth, Network, GUI 총 3개의 Repository로 나눠 submodule 형식으로 관리를 하거나, Swift Package Manager(이하 SwiftPM)로 프로젝트를 관리를 할 생각이 있었지만 모두가 Git을 아는게 아니였고, Private Repository로 관리를 해야 했기에 ssh-key나 gpg-key를 등록을 해야하는 등 공부 하면서 진행하기엔 내용이 너무 많아 제외 하였다.

그래서 하나의 Repository로 만들게 되었고, xcworkspace(Visual Studio 와 같은 solution 파일)를 이용해 프로젝트를 연결할 계획을 세웠었다.

하지만 아래의 이미지 처럼 BCBluetooth나, BCNetwork를 iOS 장비에 올려서 테스트 하기 위해서는 새로운 프로젝트를 만들어서 테스트를 하고 병합 하는 형식으로 했야 한다.
그렇게 된다면 xcworkspace가 충돌이 나게 되어 버리므로 현재 Branch를 유지 하고 개발을 하는 방법으로는 GUI Branch로 가서 Scheme를 이용해서 GUI 테스트 하도록 하였다.

![프로젝트 설정](/assets/img/post/2021-05-27-projectfile.png)

---

# 개발 이전 상태

라이브러리 2개(Network, Bluetooth)와 실행이 가능한 프로젝트 1개(GUI)로 구성하여 개발을 진행 하고 있었으며, Network 경우에는 XCTest 라이브러리를 이용하여 Unit Test를 통해 코드 무결성 또는, 서버와 통신에서 정상적으로 동작하는지 확인을 하여 결과를 확인 하였다.

그렇지만 Bluetooth 경우에는 데이터가 들어오고, 해당 데이터가 정상적으로 파싱(Parsing)이 가능한지 확인만 가능하고, 장비와의 연결 테스트, 신호 강도, 장비에서 데이터가 정상적으로 넘어 오는지 등, 장비와 관계된 테스트가 불가능 하다 보니 GUI에 연결 한 뒤 실기기에 올려 테스트 하는 방식이 반드시 필요 했었다.

처음에는 ble-test라는 프로젝트에서 자체적으로 만들어 진행을 하였지만, 6월 이전 까지는 적용이 완료가 되기 원하고 있던 상황 이였다.

---

# 현재 상태

현재는 ble-test라는 프로젝트를 GUI 프로젝트에 옮긴 뒤 Scheme를 이용하여 자동적으로 빌드 결과가 다르게 하였다.

![프로젝트 설정](/assets/img/post/2021-05-27-schemestate.png)

BoomCare라는 프로젝트로 실행 하게 될 경우 GUI 프로젝트가 빌드가 되면서 결과를 출력 하고, BluetoothBoomCare로 하게 될 경우 ble-test가 빌드가 된다.

![프로젝트 설정](/assets/img/post/2021-05-27-nowprojectfile.png)

즉 BoomCare Scheme로 실행 하게 되면 Entry Point(진입점)은 Main.Storyboard가 되면서
최초 실행 화면이 나오게 되며, BluetoothBoomCare로 실행하게 되면 Entry Point가 BLEMain.storyboard가 된다.

---

# 방법

방법 자체는 인터넷에 자세한 설명이 있기도 하며, Xcode일 때에만 적용이 가능한 내용이기 때문에 간략하게 설명 하고자 하고, 이미지로만 대체 합니다. 맨 아래에 이미지로 구성이 되어 있습니다.

---

# 생각 그리고 후기

처음에는 Visual Studio에서 여기에 있는 Debug와 Release가 기본적으로 있는 반면, Unreal Engine 4 에서 만들어 주는 솔루션 파일에서는 다양한 Scheme가 있는걸 1차적으로 확인을 하였습니다.

그래서 이걸 어디에 적용을 시켜야 할까 라는 과거에 고민을 했었고, 현재 연구과제를 진행 하면서 여러 사람이 같이 개발을 할 때 적용을 하거나, 하나의 프로젝트 이거나, 다수의 프로젝트 일 때 빌드 가이드 라인을 제공 할 수 있겠다 라는 생각이 들었습니다.

예를 들어 거대한 프로젝트가 있다고 하였을 때, 테스트 서버와 라이브 서버를 빌드 하기 위해서 Flag를 이용해서 매번 수정을 해야 하는것이 아닌, Scheme를 이용해서 자동적으로 빌드가 되는 순서를 정하는, 즉 Continuous Integration(CI) 작업을 할 수 있다는 걸 알게 되었습니다.

과거 실 기기 통신 하기 위해서 코드와, 시뮬레이터와 통신하는 코드를 if문으로 나눠서 처리를 했었습니다.

```cs
#if ISDEBUG == TRUE
    실기기 관련된 초기화
#else
    시뮬레이터 관련된 초기화
#endif
```

하지만 Scheme를 이용해서 Test Scheme와 Live Scheme를 만들게 된다면자동적으로 처리가 되므로 매우 편리 할것 같습니다.

---

# 방법에 관한 이미지

![이미지](/assets/img/post/2021-05-27-dude1.png)
![이미지](/assets/img/post/2021-05-27-dude2.png)
![이미지](/assets/img/post/2021-05-27-dude3.png)
![이미지](/assets/img/post/2021-05-27-dude4.png)
![이미지](/assets/img/post/2021-05-27-dude5.png)
![이미지](/assets/img/post/2021-05-27-dude6.png)
![이미지](/assets/img/post/2021-05-27-dude7.png)
![이미지](/assets/img/post/2021-05-27-dude8.png)
