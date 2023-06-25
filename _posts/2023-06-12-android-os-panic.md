---
layout: post
title: 메모리 동적 할당을 이용한 안드로이드 커널 패닉 유도     
author: piorosen
categories: [Blogging, Develop]
tags: [automate, ci, github-action, absenteeism, tardiness, ftp]
hide_title: false
---

# 개요

안드로이드 운영체제에서 네이티브로 동작하는 프로그램을 개발해야한 상황이였다. 이때, 네이티브하게 동작한다는 것은 어플리케이션을 실행하는 것이 아닌, 실행 바이너리를 ADB 로 실행하는 것을 의미한다. 

여기서 이게 가능한가..? 라는 의문이 든다면 리눅스의 구조와 커널 구조에 대해서 이해가 더 필요하다. 간단하게 깨달음을 준다면 우분투에서 프로그램을 설치 할 떄 사용하는 `sudo apt install *` 통한다. 이때, 놀랍게도 `sudo` 는 프로그램이며, 란 `ls`, `cat`, `cp` 또한 `프로그램`이다. 그 말은 즉, 누군가가 사전에 프로그램을 설치한 것이다. 그러므로 `ls`나 `cp`와 같은 프로그램을 빌드 하고, adb에서 호출 또한 가능할 것이다.

우선, [[NDK를 이용하여 C++로 웹 서버 구축하기]](/posts/android-native-app/) 위의 내용이 잘 이해가 되지 않는다면 해당 글을 읽고오는것은 추천한다.

# 환경 설명

우선 테스트 환경을 2가지로 정의하고, 실험을 진행해본 뒤 결과를 분석해본다.

1. ADB를 이용하여 바이너리 실행
2. 안드로이드 JNI를 통해 C++ 코드 실행

실험은 프로그램을 강제적인 오류를 발생 시키기 위해 메모리를 무한정 할당 시도하여 메모리 할당 실패시키는 방식으로 진행한다. 

# 실험 코드 및 환경 정의

> Android OS : 10<br> 
> 퀄컴 스냅드래곤 865 개발자 보드 <br>
> C++ : 14, [NDK r25b](https://dl.google.com/android/repository/android-ndk-r25b-linux.zip?utm_source=developer.android.com&utm_medium=referral), (컴파일러 : aarch64-linux-android21-clang++)

```cpp
#include <vector>
#include <iostream>
#include <random>

static std::vector<double> data;

int main(int argc, char** argv) {
    std::cout << "RUN!\n";    
    std::random_device rd{};

    double random[1024];
    for (int i = 0; i < 1024; i++){ 
        random[i] = rd();
    }

    for (int j = 0; ; j++)
    { 
        for (int i = 0; i < 1024 * 1024; i++) { 
            data.push_back(random[i % 1024]);
        }
        std::cout << (j + 1) * (sizeof(double)) << " MB\n";    
    }
    return 0;
}
```

# 실험 결과

아래의 영상을 확인해 보시면 4GB 정도 메모리를 할당한 다음 안드로이드가 강제로 죽는것을 볼 수 있습니다.

[![](/assets/img/post/2023-06-25-poc.jpg)](https://youtu.be/Ici8rg1EVog)

