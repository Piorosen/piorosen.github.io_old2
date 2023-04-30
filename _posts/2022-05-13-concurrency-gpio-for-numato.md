---
layout: post
title: numato의 Serial와 GPIO 통신 모듈 사용기
author: piorosen
tags: [gpio, numato, usb, serial]
categories: [Blogging, Develop]
hide_title: false
---

# 개요
연구과제를 진행 하던 중 특정 장비를 컨트롤 및 관리를 해야 했다. 하지만 특정 장비와 통신을 하기 위해서는 GPIO 통신만 가능했기 때문에 일반 윈도우 PC에서 GPIO를 주고 받는 모듈을 이용해야 했다. 
그래서 기업에서 주로 사용하고 있는 numato사의 32USBGPIOModule를 활용하여 윈도우 PC에 연결하여 사용했다. numato사의 USBGPIO 모듈을 사용하면서 발생 했던 문제, 내용을 기록하였다.

|GPIO Module Demo 프로그램|
|:---:|
|![GPIO 이미지](/assets/img/post/2022-05-13-gpio-demo.png)|

# 신기한 구조

정말 신기한 구조 였다. GPIO는 특정 핀을 HIGH, LOW로 하면서 다른 장비와 0과 1이란 시호를 주고 받는다. 특히 GPIO 핀은 신호를 주고 받는 방식에 따라 다르지만, 전압에 따른 통신이다. 하지만 시리얼 통신은 GPIO와 비슷하지만, 특정 핀을 Baudrate 만큼 HIGH, LOW로 하면서 오류 정정 제어가 가능한 프로토콜 및 통신 이다. 그렇기 때문에 시리얼 신호를 GPIO 신호로 완전히 바꾸는것은 어렵다. 그래서 numato사에서 만든 32USBGPIOModule은 매우 신기한 구조로 동작하고 있었다. 위의 numato사에서 만든 데모 프로그램을 보게 된다면 느낌적인 느낌적으로 아! 나는 중간 매개체와 통신을 통해 GPIO를 컨트롤 할 수 있겠구나! 라는 생각이 들었다. 그 생각은 들어 맞았으며 코드를 보면서 해독했다.

일반 PC에서는 32USBGPIOModule 기기와 Serial 통신을 하고, 32USBGPIOModule가 다시 한번 GPIO 핀을 컨트롤 하는 장비, 또는 스스로 명령어를 분석하여 수행 한다. 조금 단순하게 이야기를 하자면 항상 PC와 연결이 되는 아두이노 보드가 있고, 아두이노 보드에서 시리얼 통신으로 PC로 부터 데이터를 받게 되면 GPIO 핀을 HIGH나 LOW로 바꾸고, 펌웨어 버전이나 IOMASK등 다양한 명령어를 실행하는 보드가 있는것이다.

이러한 구조 때문에 Serial 통신으로 명령어를 날리면 GPIO 핀이 응답하는 구조이다.

# 사용한 명령어 또는 사용법

자세한 도큐먼트는 [[numato사의 도큐먼트]](https://numato.com/docs/32-channel-usb-gpio-module-with-analog-inputs/#the-commands-set-2)에서 명령어 세트를 확인 할 수 있다.
내부적으로 실질적인 통신은 시리얼 통신이기 때문에 시리얼, COM 포트를 개방 해야하고, 그 이후에 명령어는 "\n"이 아닌 "\r" 이다. 그리고 값을 지정하는 set, clear, 값을 읽어오는 read가 있다.

```cpp
gpio set 1 // 1번 GPIO 핀을 HIGH 로 설정함.
gpio clear 2 // 2번 GPIO 핀을 LOW로 설정함.
gpio read 9 // 9번 GPIo 핀의 값을 읽어옴.

// 32채널 이므로 10 이상의 핀의 경우에는 10 이 아닌 A, B로 나타냄.
gpio set V // 32번 GPIO 핀을 HIGH로 설정함.
```

그 외의 데이터들은 모두 GPIO와 관련이 되어 있지만 일괄 변경, 일괄 변경 기능이다. 

# 마지막

처음에 GPIO 핀 테스트를 8번핀까지 테스트를 진행을 했었었다. 그래서 처음에는 GPIO 핀이 숫자 0~9 까지의 범위 이므로 정상적으로 동작 했지만, 9 이후에는 당연히 10, 11 이라 넘어갈것이라 생각하고 코드를 작성 했던 것이 문제 사항이였다. 그래서 실제 미팅에서 16번핀, 18번핀을 사용하니 문제가 발생하고 말았다. 해당 문제를 찾기 위해서 기업체 연구실에서 1시간 정도 전압 측정기로 HIGH로 넘어오는지 LOW인지 값이 정상적으로 읽어지는지 확인하는 디버깅 작업이 진짜 사람 머리를 피를 말리는 상황이였다.

다음주에 간 작업은... 교수님과 박사님과 총 4분이 2시간 정도 디버깅 하는걸 지켜본건 비밀이다...
