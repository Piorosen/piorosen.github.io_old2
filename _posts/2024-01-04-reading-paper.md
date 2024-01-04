---
layout: post
title: PyGuard 파이썬의 보안 취약점 검사 논문 읽기
author: piorosen
categories: [Paper, Review]
tags: [security,paper,review,python,cpython,vulnerablilites]
hide_title: false
---

# 개요

논문 명은 `PyGuard: Finding and Understanding Vulnerabilities in Python Virtual Machines` 이며 읽게된 계기로는 심심해서 읽게된 논문이다. 그렇게 IF나 유명하진 않지만 `PyArmor`라는 프로젝트 명이 생각나지 않아서 `PyGuard`인가..? 하다가 찾게 된 논문이다. 

해당 논문은 ISSRE 학회 2021년 컨퍼런스에서 발표된 내용이며, 저자가 이야기 하는 기여점은 `CPython`에 대한 보안취약점을 탐색하고, 보안 취약점에 대해서 분류하는것이다. 이를 위해 `파이썬 3.0` 버전 부터 `3.9` 까지 총 10개의 버전을 수행하였다.

그리고 추가적으로 파이썬 버전의 취약점을 검사하기 위해 개발된 소프트웨어에 대해서 설명하고, 어떻게 구현하였는지와 추가적인 확장 도구들을 쉽게 통합할수 있었는지 설명합니다. 본 논문에서 제안한 `PyGuard`의 시스템 구조는 아래와 같습니다. 

이때, 가장 의문스러운 부분은 `CPython Source Files`와 `Native Code`의 차이점이 궁금하지만 본 문에서 설명한 내용은 거진 2문장으로 설명이 끝나다 보니, 의마가 제대로 전달이 되지 않는것 같았다.

- 필자의 생각으로는 `CPython Source Files`는 주석과 도큐먼트 파일이 모두 포함된 코드를 의미하는 것이며, `Native Code`는 오직 순수 C 파일만 있는 구조가 아닐까 라는 생각이 됩니다. 처음에는 `outputs the target CPython native code`라 설명이 되어있어 빌드 된 코드를 의미하는 것이라 생각하였지만, Architecture에 `Security Plug-in module`에 CppCheck가 있어 아닌 것이라고 생각하게 되었습니다.

`takes the CPython source code as input and outputs the target CPython native code, according to the configuration file.`

`The input processing module will extract both the native code and the native method interface code, from a given Python virtual machine source code.`

![](/assets/img/post/2024-01-05-02.png)


# 결론

버퍼 오버플로우나, 메모리 검사, 포맷 공격, 문법 검사 등 다양한 공격 기법을 탐지하는 모듈을 활용하여 보안 취약점을 검사하였다. 해당 논문에서는 

`These system can eliminate buffer overflows, format string attacks and many memory management errors in C programs.` 

로 이야기 하였다. 

그 결과 파이썬의 버전별로 보안 취약점 검사를 수행한 결과로는 파이썬 3.0 버전 부터 3.2 버전까지 탐지된 보안 취약점이 점진적으로 감소되는 모습을 보여주었다. 하지만 `CPython 3.3` 버전에서는 탐지된 취약점이 갑작스럽게 증가가 된 모습을 보여준다. 이는 `CPython 3.3` 버전에 새로운 기능이 추가가 되면서 생겼다라고 이야기가 되어지고 있다. 그래서 새로운 버전이 나온 이후에 나온 `CPython 3.4` 버전 이후 부터는 안정성이 추가가 되면서 보안 취약점이 낮아지는 모습이 나타난다.   

![](/assets/img/post/2024-01-05-01.png)

CPython의 버전별로 증가가 되는 코드의 양과 그에 대한 보안 취약점에 대한 그래프 모습이다.

# 한계점

해당 논문은 다음 작업을 위해 연구된 내용다 보니 아직 완벽한 내용이나 다른 사람들에게 기여점이 무엇인지 명확하게 나타내고 있지 않습니다. 그래서 `PyGaurd`에 대한 아키텍처를 나타내고, 파이썬의 보안 취약점 검사를 수행한 결과를 나타내고 있습니다.

`It should be noted that this work represents the first step towards finding and understanding vulnerabilities in real-world Python virtual machines, by constructing a new software prototype PyGuard.`

여러 확장 도구로 인하여 많은 수의 경고가 거짓 양성으로 나타날 수가 있다는 단점지만 이에 대해서 알릴수가 없다는 단점이 있습니다.

# 결과

본 문에서는 8개의 취약점 패턴으로 나눴고, 그 중 가장 많았던 취약점은 오버플로우와 메모리 오류가 가장 많았다고 서술한다. 이러한 이유는 파이썬과 C언어간의 타입 차이로 인해 크다. 또한 파이썬에는 정수형 이란 타입이 1 종류 밖에 없지만 C언어의 경우에서는 10개가 넘기도 하며 숫자의 범위 차이 또한 있기 때문이다.

![](/assets/img/post/2024-01-05-03.png)

# 관련 연구

관련 연구로는 보안 취약점 검사에 사용되는 정적 도구와 동적 분석 도구에 대한 연구에 대해서 설명하였고, `Security Plugin`에 사용된 프로젝트에 대해서도 설명하기도 하였다. 

관련 연구에 대해서 모두 서술하고 어떻게 동작하는지에 대해서 자세하게 설명하것 보다 나열하는것이 더 괜찮을 거라 생각하게 되었다. 이는 기법에 대해서는 새로운 레퍼런스 논문을 보거나 다른 글의 자료를 찾는것이 더 좋을거라 생각하였기 때문이다.

* PyGuard에서 사용된 기법들

overflows and formatted string 검사 : FlawFinder, CppCheck, TscanCode<br>
static analysis tools: Coverity, Checkmarx, or Fortify

* native code security 관련 연구들

CCured, Cyclone

* Buffer overflow:<br>

polymorphic SSP (P-SSP), 코드 교체 기법, 딥 러닝을 통한 취약점 검사

* Memory errors:

동적 유형으로 지정된 메모리 검사, 동적 메모리 오류 검사, 동적 실행파일 변환, 실행 파일과 콜 스택 분석 기법, 메모리 접근 강도 분석


# 개인적인 견해

연구에 대한 필요성과 타당성 설명쪽에서 1문단으로 마무리를 짓는것이 조금 의아하기도 했다. 그리고 구현 논문에 대해서는 꽤 많은 설득과 빌드업이 중요하다는것을 알게 되었다. 히히 부럽다.. 영어 잘해서... 진짜로 심심해서 읽었기도 하며, 재미있었다.

`However, a major limitation of these work is that they only studied vulnerabilities in common native code, but does not discuss the vulnerabilities and patterns in a virtual machine setting. Furthermore, these work don’t discuss the taxonomy of vulnerabilities.`
