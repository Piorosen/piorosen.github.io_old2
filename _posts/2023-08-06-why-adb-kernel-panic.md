---
layout: post
title: 안드로이드에서 메모리 동적 할당을 이용하면 커널 패닉이 발생하는 이유
author: piorosen
categories: [Blogging, Develop]
tags: [automate, google, ndk, issue, github, memory, OOM, OOM-Killer]
hide_title: false
---

# 개요

우선, 해당 글의 경우 이슈의 발달 원인인 [[[관련 주소]]](/posts/android-os-panic/)에서 한번 확인해 주시길 바랍니다. 먼저, 이슈의 발달된 내용은 메모리를 무한정 할당할 경우, System UI가 갑작스럽게 사망하게 되면서 시스템이 정지한 듯한 모습을 보여주는것이 문제의 발달 계기 입니다. 

해당 문제를 재현하고 시현하기 위해서, 단순하게 자바 어플리케이션을 활용하여 앱 개발하는 것이 아닌, NDK 즉, C++을 이용하여 프로그램을 제작하여야 합니다.

따라서, 해당 문서를 읽기 위해서 기본적인 선행 지식으로 C++, 크로스 컴파일, 임베디드, CMake, Linux, glbic에 대해서 미리 알고 계셔야 합니다. 

# 시나리오

먼저, 테스트를 진행하면서, 작성한 코드의 시나리오의 경우 C++에서 vector를 생성하고 해당 컨테이너에 무한정 데이터를 삽입하는 것으로 시나리오는 끝나게 됩니다.

그리고 개발 환경과 실험 환경으로는 NDK25b와 ADB 쉘에서 실행하였습니다. 실행하게 되면 아래의 이미지 (클릭 가능합니다)와 같이 SystemUI가 강제 종료가 되고 더 이상 UI가 표기 되지 않습니다. 이를 해결하기 위해서는 재부팅이란 방법 밖에 존재하지 않습니다.

[![](/assets/img/post/2023-06-25-poc.jpg)](https://youtu.be/Ici8rg1EVog)


마지막으로 실험에서 비교군을 통해 다른 디바이스에선 어떻게, 아니면 다른 코드를 작성하였을 때는 어떻게 되는가에 대해서 정의가 필요합니다.

1. `malloc`를 통하여 변수를 생성하였을 때
2. `realloc`을 통하여 변수의 크기를 계속 늘려나갔을 때
3. 일반 x86 리눅스에서 실험하였을 때

로 크게 3가지 경우로 실험을 시작하였습니다.

# 실험 결과

1. `malloc을` 통하여 변수를 생성하였을 때
- `bad_alloc` 이란 예외가 발생하였습니다.

2. `realloc`또는 `vector`을 통하여 변수의 크기를 계속 늘려나갔을 때
- `SystemUI`와 `adb shell`이 강제 종료가 되었습니다.

3. 일반 x86 리눅스에서 실험하였을 때
- `bad_alloc`이 발생하였습니다.

# 실험 결과 이유 분석

해당 문제와 관련하여 안드로이드의 NDK 개발팀에 문의를 남겼었고, 이에 대하여 응답을 받게 되었습니다. [[해당 관련 이슈]](https://github.com/android/ndk/issues/1897)는 하이퍼 링크로 남겼으며, Android/NDK 레포의 1897번의 이슈로 남아있습니다.

(사담이긴 하지만 해당 문제를 파악하기 위해 android의 LLVM의 GLIBC 코드 분석부터 확인하였습니다 ㅎㅎ...)

응답으로 받은 결과로는 아래와 같습니다.

```
This is intended behavior, processes from adb shell have a default
oom_score_adj of -950, so the launcher will get
oom-killed before anything you run from adb shell does.
```

으로 받았으며, 이를 해석하게 되면 아래의 문장과 같이 됩니다.

```
메모리 부족하게 될 경우 우선순위에 따라서 순차적으로 프로그램을 
종료하게 됩니다. 그리고 adb shell이 SystemUI 보다 더 높은 메모리 
점유에 대한 우선순위를 가지고 있어서 발생한 문제입니다.
```

그렇다면 `realloc`과 `vector`에 계속 데이터를 삽입하였을 때에 대한 문제는 해결이 되었지만, `malloc`에 대한 문제는 아직 해결이 되지 않았습니다. `malloc`만 왜? `bad_alloc`이 발생하였는가? 는 해결이 되지 않았습니다.

그래서 이에 대해 추가적으로 질문을 해본 결과로는 아래와 같습니다.

```
That's because you're not touching the allocations, so no actual
memory pages are being allocated. You're running out of address
space or virtual memory mappings, not actual memory.
```

`malloc`과 `new`는 실제 물리적인 주소와 메모리 할당을 합니다. 하지만 `realloc`의 경우에는 물리적인 주소를 1GB 이상 요청하게 된 경우, 1GB 메모리 공간을 만드는 것은 매우 어렵습니다. 그래서, 가상 메모리란 개념을 도입하여 데이터를 관리하고 있습니다. 

그렇기 때문에, realloc은 실제 메모리의 유효공간이 남아 있으므로 추가적으로 공간 할당을 요청하게 됩니다. 그렇지만, 메모리가 부족하여 SystemUI를 강제 종료하게 되는 것입니다. 

# 요약

메모리가 부족하면, 우선순위가 낮은 순서대로 시스템을 강제 종료하게 됩니다. 이는, Linux 커널의 정책에 따라 달라지게 될 것이며, 커스텀화된 OS의 경우 또 달라지게 됩니다.

구글측에선 메모리 동적할당과 관련하여 ADB에 관련된 주석을 추가해 주었습니다.

[[Android OSP의 커밋 기록]](https://android-review.googlesource.com/c/platform/bionic/+/2638672)

[[Github 커밋 기록]](https://github.com/msft-mirror-aosp/platform.bionic/commit/0a94e1584ecb9244495ad1001f5475bc49041a70)

추가된 커밋 기록
```
 * Note that Android (like most Unix systems) allows "overcommit". This
 * allows processes to allocate more memory than the system has, provided
 * they don't use it all. This works because only "dirty" pages that have
 * been written to actually require physical memory. In practice, this
 * means that it's rare to see memory allocation functions return a null
 * pointer, and that a non-null pointer does not mean that you actually
 * have all of the memory you asked for.
 *
 * Note also that the Linux Out Of Memory (OOM) killer behaves differently
 * for code run via `adb shell`. The assumption is that if you ran
 * something via `adb shell` you're a developer who actually wants the
 * device to do what you're asking it to do _even if_ that means killing
 * other processes. Obviously this is not the case for apps, which will
 * be killed in preference to killing other processes.
```

