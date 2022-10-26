---
layout: post
title: Intel SIMD를 통한 고속 행렬 연산(기초)
author: piorosen
categories: [Blogging, Develop]
tags: [SIMD, Intel, SSE, naive]
hide_title: false
---

# 개요
Single Instruction Multiple Data(SIMD) 기술은 1회 연산에 여러개의 데이터를 병렬적으로 처리하는 기술이다. 관련된 자료는 찾아보면 엄청 많이 있으므로 생략을 한다. 간단하게 설명을 하자면 CPU의 클럭은 보통 Dynamic Voltage Frequency Scaling(DVFS)나, 내부적인 Time Slice(Task) 스케줄링 기법이 없다면 보통은 고정적이다. 즉, 반복문을 수천 수만번 반복을 한다면 특별한 사유(백그라운드 프로그램 및 L1, L2 캐시, 주 메모리의 쓰로틀링)가 없다면 Performance Monitor Unit(PMU)에서 성능 측정한다면 거의 오차가 없다. 또는 없어야 한다. 그런 제한적인 성능에서 더 많은 연산을 수행하기 위해 SIMD를 통하여 1회 연산에서 병렬처리를 할 수 있다.

# 기존 행렬 연산

기존 행렬의 곱셈은 MxN * NxK = MxK 의 값을 구하는 구조이다. 그렇기 때문에 3중 반복문을 통하여 MxK 와 N번 반복을 한다면 모든 연산을 수행 할 수 있다. 그래서 아래의 코드 처럼 3중 반복문과 곱셈과 덧셈을 통해 구현 할 수 있다. 
512x512 * 512x512 행렬의 연산을 수행하면 약 1초 이상의 시간이 소요가 된다.

```cpp
const int MAX_DIM = 512;
for (int i = 0; i < MAX_DIM; ++i) {
    for (int j = 0; j < MAX_DIM; ++j) {
        for (int k = 0; k < MAX_DIM; k++) {
            d[i * MAX_DIM + j] += a[i * MAX_DIM + k] * b[k * MAX_DIM + j];
        }
    }
}
```

#  Streaming SIMD Extensions를 통한 최적화

하지만 SIMD의 구현체 중 하나인 SSE를 활용 한다면 1 클럭에 여러번 연산을 수행할 수 있다. 현재 사용한 SSE는 128비트를 한번에 연산하는 기능이며, 16비트 Short를 곱, 덧셈 하므로, 1 클럭에 8개의 데이터를 병렬 처리 한다. 이로써, 기존 성능에 비해 8배 더 빠른 결과를 낼 수 있다.

> 문서: https://stackoverflow.com/questions/40313434/sse-matrix-matrix-multiplication

```cpp
// this is significantly better but doesn't do any cache-blocking
void matmulSSE(short* mat1, short* mat2, short* result) {
    for (int i = 0; i < MAX_DIM; ++i) {
        for (int j = 0; j < MAX_DIM; j += 8) {   // vectorize over this loop
            __m128i vR = _mm_setzero_si128();
            for (int k = 0; k < MAX_DIM; k++) {   // not this loop
                __m128i vA = _mm_set1_epi16(mat1[i * MAX_DIM + k]);  // load+broadcast is much cheaper than MOVD + 3 inserts (or especially 4x insert, which your new code is doing)
                __m128i vB = _mm_loadu_si128((__m128i*) & mat2[k * MAX_DIM + j]);  // mat2[k][j+0..3]
                vR = _mm_add_epi16(vR, _mm_mullo_epi16(vA, vB));
            }
            _mm_storeu_si128((__m128i*) & result[i * MAX_DIM + j], vR);
        }
    }
}
```

# 결과

SIMD가 아닌, Naive한 구현체로는 성능이 약 2.5초가 걸리지만, SSE로 구현한 예제는 2배 빠른 1.2초의 시간이 걸리면서 약 2배의 시간이 단축 되었다. 그리고 FastIO를 통하여 0.7초 까지 단축 할 수 있었다. 

![결과 이미지 성능비교](/assets/img/post/2022-10-05-result.png)

# 결론 글 쓰기가 매우 귀찮다...
