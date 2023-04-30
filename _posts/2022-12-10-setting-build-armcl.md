---
layout: post
title: Arm의 ComputeLibrary인 ArmCL 크로스 빌드 및 통합 과정
author: piorosen
categories: [Blogging, Develop]
tags: [arm64, arm, aarch64, cpu, armcl, cmake]
hide_title: false
---

# 개요
2022년 11월에 발표한 ["단일 ISA 이기종 멀티 코어 구조를 위한 프로파일 기반 ArmCL 최적 스케줄 탐색"](assets/paper/2022_11_단일%20ISA%20이기종%20멀티%20코어%20구조를%20위한%20프로파일%20기반%20ArmCL%20최적%20스케줄%20탐색.pdf) 학술지를 진행하면서 겪었던 내용을 정리 및 ArmCL 개발 관련 일정을 정리하였다. 우선적으로 ArmCL은 라이브러리 형식만 지원 하므로 기기에서 테스트를 하기 위해선 실행파일로 구현해야한다. 이 과정에서 ArmCL을 커스텀하고 빌드하기 위한 과정이다. 빌드가 어려운 이유는 크로스 빌드 경험, C++ 개발 경험이 부족하다면 개발 환경을 구축에 많은 시간을 투자가 필요하기 때문이다. 그래서 최대한 쉽게 설명하기 위해 작성하게 되었다.

# 설명

ArmCL에 대해 설명하기 앞서 필요한 전제 조건을 나열한다. 우선 크로스 컴파일이란 완전히 별개의 2개의 시스템이 있을 때, 다른 1개의 시스템에서 컴파일 한 결과가 다른 시스템에서 동작하는 것이다. 예시로 `Arm`환경의 `MacOS`에서 `X86`환경의 `Windows`를 위한 프로그램을 빌드하는 것이 크로스 컴파일이다.

그래서 빌드가 가능한 운영체제와, 실행 가능한 환경이 나눠지는것이다. 또한 `ArmCL`은 `CPU`, `GPU` 가속화를 지원하지만, 내부적으로 호환성을 위해 `#ifdef` 형식으로 운영체제별 구현이 되어 있습니다. 

![arm architecture](/assets/img/post/2022-12-10-armnn.png)
[출처 : Build Arm NN custom backend plugins](https://developer.arm.com/-/media/Arm%20Developer%20Community/PDF/Machine%20Learning/Machine%20Learning%20PDF%20Tutorials/Build%20Arm%20NN%20custom%20backend%20plugins.pdf?revision=100c542f-c3de-4389-8379-c17b31ea1f61&psig=AOvVaw1oRc0dKcK63TcxZD59zpds&ust=1670775943596000&source=images&cd=vfe&ved=0CBAQjRxqFwoTCNCAu8m77_sCFQAAAAAdAAAAABAE)

ArmCL은 위의 그림처럼 볼 때, NEON과 CL을 가지고 있다는 형식으로 표현이 되어 있다. NEON은 Arm CPU의 SIMD(Single Instruction Multi Data, 즉 1회 명령어로 여러개의 데이터를 처리하는 기술이다.) 기술 명이며이다. 실제로 코드를 분석하게 된다면 NEON을 랩핑하여 만든 구현체와 OpenCL을 랩핑하여 제공되는 버전 총 2가지가 존재한다. 그래서 OpenCL을 통해 실행한다면, 실행은 어떠한 환경에서든지 동작을 지원한다. 그러나 특정 CPU의 특성을 타는 NEON의 경우 동작하지 않는다.

## 빌드 관련 운영체제

1. + X86 Linux, Arm64 Linux 와 같은 리눅스 계열은 모두 지원한다.
2. + 놀랍게도 Windows 또한 지원 한다.
3. - MacOS는 지원하지 않습니다. 그래서 Docker를 이용한 가상화를 통해 실행해야 합니다.

## 빌드에 필요한 도구

1. GCC 계열의 빌드 도구
2. scons (ArmCL의 빌드 도구, CMAKE와 기능적 동일함.)
3. g++-arm-linux-gnueabi 
    g++-arm-linux-gnueabihf 
    crossbuild-essential-arm64 
    crossbuild-essential-armhf (빌드를 위한 필수 도구)

# 도커를 통한 빌드 설명

도커를 통해 만약 빠른 빌드를 원한다면 아래의 명령어를 수행함으로써 해결 할 수 있습니다. 이때 빌드된 결과물은 ArmCL 22.05 버전이므로 아래의 `git checkout tags/v22.05`를 수정함으로써 변경할 수 있을것 이라 생각합니다. 솔직한 감상으로는 `docker` 빌드 결과물을 제공함으로써, 쉽게 접근 할 수 있습니다.

`docker run --rm -it -v $pwd:/build armcl bash` 

```docker
FROM ubuntu:20.04 AS builder

ENV DEBIAN_FRONTEND=noninteractive \
    TZ=Etc/UTC
    
RUN apt update && DEBIAN_FRONTEND=noninteractive TZ=Etc/UTC apt -y install locales tzdata

RUN apt install -y \
    scons \
    g++-arm-linux-gnueabi \
    g++-arm-linux-gnueabihf \
    git \
    crossbuild-essential-arm64 \
    crossbuild-essential-armhf

ENV BUILD_CORE=16 \
    BUILD_DEBUG=1 \
    BUILD_NEON=1 \
    BUILD_OPENCL=1 \
    BUILD_EXAMPLE=1 \
    BUILD_ARCHITECTURE=arm64-v8a

RUN git clone "https://github.com/Arm-software/ComputeLibrary" && \
    cd /ComputeLibrary && \ 
    git checkout tags/v22.05 && \
    scons Werror=1 debug=${BUILD_DEBUG} asserts=0 \
          neon=${BUILD_NEON} opencl=${BUILD_OPENCL} \
          examples=${BUILD_EXAMPLE} \
          arch=${BUILD_ARCHITECTURE} -j${BUILD_CORE} && \
    mkdir /build && \
    mv /ComputeLibrary/build / && \ 
    rm -rf /ComputeLibrary && \
    rm -rf /build/*.a && \
    rm -rf /build/src /build/tests

ENV LD_LIBRARY_PATH=/build
```

# 개발 환경 구축

도커를 통해 ArmCL을 빌드에 성공하였다면, 아래의 명령어를 수행하여 CMAKE로 바로 개발이 가능한 프로젝트를 사용할 수 있다. ArmCL의 빌드된 파일을 external에 넣어주는것으로 ArmCL을 개발할 환경 구성이 끝난것이다. 만약 크로스 컴파일을 해야한다면, `g++`이 아닌 `aarch64-linux-gnu-g++` 로 변경하면 바로 사용이 가능하다. 만약 일부 커널 버전이 낮아, gcc 7버전이나, 그 이하 버전이 필요한 경우 마찬가지로 아래의 다운로드링크로 나타낸다.

> `aarch64-linux-gnu-g++`의 7.5.0 버전 다운로드
> wget https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz
> 템플릿 구성
> git clone https://github.com/Piorosen/ArmCL-CMake-Template

