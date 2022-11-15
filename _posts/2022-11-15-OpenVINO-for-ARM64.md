---
layout: post
title: Arm64에서 동작하는 OpenVINO 빌드 방법 및 통합 과정
author: piorosen
categories: [Blogging, Develop]
tags: [openvino, arm, aarch64, myriad, neural-compute-stick-2, ncs2, intel, npu]
hide_title: false
---

# 요약

성공하고, 정상적으로 동작하는것을 확인하였을 때의 신남은 말로 할 수 없었습니다. 또한 C/C++과 CMake, Cross Compile, 툴체인 등 다양한 선행 지식이 있었음에도 불구하고 성공하기 까지 약 4일 정도 걸렸습니다. 거진 시간으로 따지자면 약 50~60시간 정도 투자를 한 느낌입니다. 그리고, OpenVINO는 오직 Intel만 공식적으로 지원하므로, ARM에서 NCS2를 활용하는것은 초심자에게 많이 어려울 것이라 생각합니다. 이유로 첫째, 문서화가 되어 있는 것은 Rasberry PI만 있어 Arm기반 Linux를 위해 크로스 빌드와 실행 관련된 자료가 미비해 어렵습니다. 둘째, 런타임 도구와, 확장 도구(예, 모델 양자화, 연산 최적화 등)은 오직 파이썬만 사용이 가능하므로, C++ 라이브러리와, Cython과 연동하는 과정이 필요 합니다. 그 외에 다양한 어려움이 있어 초심자에겐 어려우므로 내용을 정리하여 기록합니다.

# 결과

OpenVINO 모델 형태 분석|AlexNet 1회 추론 속도
:---:|:---:
![모델 형태 분석](/assets/img/post/2022-11-15-1.png)|![모델 형태 분석](/assets/img/post/2022-11-15-2.png)


# 네이티브 빌드

간략하게나마... 빌드하기 위해 확인한 문서를 나열을 합니다.

1. [어떻게 Odroid에서 OpenVINO를 사용할 수 있습니까?](
https://magazine.odroid.com/article/creating-a-vision-application-in-low-power-situations-using-openvino-and-opencv-with-the-odroid-c2/)
2. [X86에서 Arm64용 OpenVINO를 빌드하기 위한 논의](https://github.com/openvinotoolkit/openvino/issues/8343)
3. [Python API에서 OpenVINO를 활용하기 위한 방법](https://www.intel.co.kr/content/www/kr/ko/support/articles/000057448/software/development-software.html)
4. [OpenVINO의 모델을 최적화 하기 위한 'mo'가 없습니다.](https://github.com/openvinotoolkit/openvino/issues/11150)

Odroid는 Arm 보드 모델 중 하나 입니다. 이와 같이 문서를 확인하면서 쉽게 OpenVINO를 ARM에서 사용하기 위한 명령어를 작성 하였습니다.

> 1. 운영체제 : Linux linaro-alip 4.4.194 #4 SMP Tue Feb 15 03:07:56 UTC 2022 aarch64 GNU/Linux
> 2. 프로세싱 유닛 : RK3399Pro(A53 *4, A72 *2, Mali GPU, Rockip NN)
> 3. 메모리 : 4GB
> 4. gcc/g++ : 8.3.0
> 5. OpenVINO : 2022. 03 (master branch)

추가적인 내용이 필요하다면, 메일을 통해 회신해 드리겠습니다. 글을 작성하는 필자는, 크로스 컴파일을 수행은 지속적인 문제가 발생하여 포기를 하였습니다. 그래서 보드에서 직접 빌드하는 네이티브 빌드를 진행을 하여 문제를 해결 하였습니다. 라즈베리파이가 아닌, [Asus Tinker Edge R](https://tinker-board.asus.com/product/tinker-edge-r.html) 보드에서 빌드 하였을 때 4시간이 걸렸음을 확인해 주시길 바랍니다.

```sh
git clone --recursive https://github.com/openvinotoolkit/openvino

GCC_COMPILER=/usr/bin/aarch64-linux-gnu
CC=$GCC_COMPILER-gcc
CXX=$GCC_COMPILER-g++

if ! [ -d "OpenVINO/build" ]; then
   mkdir OpenVINO/build;
fi

cd OpenVINO/build && \
cmake .. \
   -DCMAKE_BUILD_TYPE=Release \
   -DCMAKE_C_COMPILER=$CC \
   -DCMAKE_CXX_COMPILER=$CXX  \
   -DCMAKE_CXX_FLAGS="-march=armv8-a" \
   -DTHREADS_PTHREAD_ARG="-pthread" \
   -DCMAKE_TOOLCHAIN_FILE="../cmake/arm64.toolchain.cmake" \
   -DCMAKE_INSTALL_PREFIX=components/OpenVINO \
   -DENABLE_MKL_DNN=OFF \
   -DENABLE_CLDNN=OFF \
   -DENABLE_GNA=OFF \
   -DENABLE_SSE42=OFF \
   -DENABLE_AVX512F=OFF \
   -DTHREADING=SEQ \
   -DNGRAPH_PYTHON_BUILD_ENABLE=ON \
   -DNGRAPH_ONNX_IMPORT_ENABLE=ON \
#    -DENABLE_PYTHON=ON \ ㅖPYTHON 과 관련된 실행 파일 및 종속성 관리 부탁드립니다.
#    -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE} \
#    -DPYTHON_LIBRARY=${PYTHON_LIBRARY} \
#    -DCYTHON_EXECUTABLE=${CYTHON_EXECUTABLE} \
#    -DPYTHON_INCLUDE_DIR=${PYTHON_INCLUDE_DIR} \
    && make -j$(nproc --all)
   echo "plz, insert ~/.bashrc under text"
   echo "source $(pwd)/components/OpenVINO/setupvars.sh" 
   echo "export LD_LIBRARY_PATH=$$LD_LIBRARY_PATH:./link" 
```

네이티브 빌드를 수행은 이것으로 마무리가 되며, ```$(pwd)/components/OpenVINO```의 폴더를 확인한다면 [[OpenVINO Archive]](https://storage.openvinotoolkit.org/repositories/openvino/packages/2022.2/linux/)를 통해 미리 빌드된 라이브러리와 똑같은 형태의 구조를 확인 할 수 있습니다.

빌드에 성공하였다면 그 이후엔 크게 어렵지 않다. 또는 필자가 직접 빌드한 Armv8a용 라이브러리를 사용하고자 한다면 [[OpenVINO 22.03 라이브러리]](https://github.com/Piorosen/HeterogeneousPU/releases/download/OpenVINO-2022-03/OpenVINO.tar.gz)를 다운로드 받을 수 있다.

# 빌드 이후 프로젝트 통합

그 이후에는 OpenVINO 에서 제공 되는 [[어떻게 프로젝트에 통합합니까?]](https://docs.openvino.ai/latest/openvino_docs_OV_UG_Integrate_OV_with_your_application.html) 란 문서를 통해 개발을 진행 하면 됩니다.

명령어로서는 CMakeLists에 추가한 뒤, 빌드가 정상적으로 되는지 확인을 통해 통합이 정상적으로 이뤄졌는지 알 수 있습니다.ㄴ

```cmake
find_package(OpenVINO REQUIRED)

...


target_link_libraries(${PROJ_NAME} openvino::runtime)

```
