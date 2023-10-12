---
layout: post
title: C++언어에서 런타임에 코드를 생성하여 실행(JIT with C++)
author: piorosen
categories: [Blogging, Compiler]
tags: [ssh, vpn, ssh-over-vpn, ssh-client, dns, internal-network]
hide_title: false
---

# 개요

그럴 일은 없겠지만, 이 글을 만약에 검색해서 읽고 있다면 무언가 잘못된 길이 아닌지 다시 한번 확인이 필요한 글일거라 생각한다. 놀랍게도, 실행중에 `코드`를 생성하여 실행하는 것은 가능하나, 정상적인 개발 환경에서는 쓰이는 일이 없기를 바랍니다.

우선, 런타임에 코드를 생성하여 실행하는 구조는 많이 찾아볼 수 있다. 대표적으로 C#의 .Net Framework나 JAVA의 JVM, Dalvik, ART가 있다. (중간 언어 Byte Code를 어셈블리로 만든 다음 실행 시키기 때문이다. 명심해야하는 부분은 이는 모두 프로그램이란 점이다.) 더 신기한 부분은 .NET Runtime과 JVM은 모두 C++로 구성이 되어 있다는 점이다.

[.Net Runtime](https://github.com/dotnet/runtime), [Open JDK(Mirror)](https://github.com/openjdk/jdk) 자세히 보면, C/C++, ASM의 비율이 약 20% 라는 것을 알 수 있다. 따라서, 이전에 이미 사례에서 C++을 이용한다면 런타임에 코드를 생성하여 실행하는 것이 가능하다는 것을 알 수 있다.

또 다른 예로는 딥러닝 가속 라이브러리인 [XNNPack](https://github.com/google/XNNPACK) 또한 내부에 JIT이 포함되어 있다. 그렇다면, 이제 가능성은 충분히 이해하였으므로 실제로 코드로 접근해보자.

# 코드

```cpp
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/mman.h>

int main() {
	unsigned char code[] = {
            0xb8, // mov
            0x00, // NOP. "nop" does nothing work.
            0x00,
            0x00,
            0x00,
            // +5
            0x83, // add == 0x83. 0x83 is assembly of x86.
            0xC0,
            0x05,

            // +5
            0x83, // add
            0xC0,
            0x05,

            // +5
            0x83, // add
            0xC0,
            0x05,

            0xc3, // ret
	};

	int num = 10; // Change! this is value for testing.

	memcpy(&code[1], &num, 4);
	void *mem = mmap(NULL, sizeof(code), PROT_READ | PROT_WRITE, MAP_ANON | MAP_PRIVATE, -1, 0);

	memcpy(mem, code, sizeof(code));
	mprotect(mem, sizeof(code), PROT_READ | PROT_EXEC);

	int (*func)() = (int(*)()) mem;

	printf("Result is: %d\n", func());

	return 0; // success
}
```
[(코드 원본 및 출처)](https://github.com/BaseMax/simple-jit-compiler/blob/main/jit.c)

각각 순서대로 접근하여 확인한다면, `unsigned char code[]`에서 x86 어셈블리 명령어로 실행할 명령어를 작성합니다. 이때, `0x00`의 의미는 NOP으로 아무것도 수행하지 않는 명령어 입니다. 주로 쓰이는 이유는, 컴퓨터는 N 바이트 만큼 읽어서 명령어를 수행하므로 전체 명령어의 사이즈를 맞춰야 빠르고 정확하게 동작합니다. (또는 맞추지 않으면 명령어가 비정상적으로 동작할 수 있습니다.) 그래서 아무 의미 없는 코드를 삽입하여 `code` 변수의 사이즈를 맞춰주는 것 입니다. `code`의 사이즈는 `16 Byte` 입니다.

이후 `code[1]`번지에 상수 값을 삽입하고, 명령어를 실행할 수 있는 권한을 부여하는 코드인 `mprotect`와 `mmap`가 있습니다. 이후, PC(프로그램 카운터)의 위치가 명령어가 있는 곳으로 이동 및 가르킬수 있도록 함수 포인터를 사용하는 과정입니다.

# 결과

실험 결과로는 당연하게 10이 들어갈 경우 15가 더해진 25가 나오는 것을 알 수 있다. 이를 통해 어셈블리를 런타임에 생성하여, 실행할 수 있으므로 뭔가 재밌는 일거리나 논문거리가 나올것 같다.

![hihi](/assets/img/post/2023-10-13.png)
