---
layout: post
title: C++에서 OS와 Compiler 판별
author: piorosen
tags: [c++, predefine, terminal, multiplatform]
hide_title: false
categories: [Blogging, Develop]
# feature-img: "assets/img/feature-img/"
# feature-img: "http://placehold.it/700x300"
# thumbnail: "assets/img/thumbnails/feature-img/"
# thumbnail: "http://placehold.it/200x200"
---

# 개요
과거 C++을 이용해서 Terminal Game Engine 을 개발 하려 했던 적이 있다. C++의 가장 큰 장점으로 생각하는 부분은 바로 멀티 플랫폼이라는 점 이였다.
어떠한 환경이여도 C++을 빌드가 가능 하며, 빌드만 가능하다면 운영체제나, CPU를 신경 쓰지 않고 어셈블리가 나오기 때문이다.
그래서 개발 하던 도중 Windows에서만 사용 가능한 기능, Linux/MacOS에서 가능한 기능을 분리해서 [Graphics Abstarction Layer](https://engineering.linecorp.com/ko/blog/line-ar-rendering-engine-yuki-elsa/) 에서 추상 API를 제작 하고 각각 OS에서 기능을 구현하는 방식으로 구현이 필요하다고 판단 했습니다.
그래서 추후에 추상 API로 구현이 되어 있으므로 OpenGL, DirectX, Metal등 다양한 API로 호출이 가능하게끔 하고자 목표도 있었으며, 첫번째 목표로는 OS를 통합하는 GAL을 구현하는것 이였습니다.

---

# 동작

MSVC나 GCC와 같은 컴파일러는 소스 코드를 컴파일 하기 전에 OS나, Compier 버전을 #define 으로 제공을 해주고 있다.

- [MSVC 컴파일러](https://docs.microsoft.com/en-us/cpp/preprocessor/predefined-macros?view=msvc-160)
- [GCC 컴파일러](https://gcc.gnu.org/onlinedocs/cpp/Common-Predefined-Macros.html)
- [Clang 컴파일러](https://clang.llvm.org/docs/LanguageExtensions.html#builtin-macros)

```shell
$ clang -x c /dev/null -dM -E
```

- [그 외 판별 하는 법](https://blog.kowalczyk.info/article/j/guide-to-predefined-macros-in-c-compilers-gcc-clang-msvc-etc..html)

C/C++언어에서 지원 해주는 pre-define 기능을 이용하여 OS와 Compiler를 인식하여 사용하기 편리하게 만든다.

---

# 코드

개발을 진행하면서 OS, Compiler만 판별하여 define 코드를 만드는것이 아닌 런타임에 분기가 필요하거나 코드적으로 필요한 경우도 지원을 하기 위한 코드도 작성을 하였다.

```cpp
// OS 를 판별 하기 위한 enum
enum class OperatingSystem {
    Windows,
    Linux,
    MacOS,
    Others
};
// Compiler 종류를 판별 하기 위한 enum
enum class KindCompiler {
    MSVC,
    MingW,
    GNUC,
    Others
};
// 프로그램의 실행 Architecture
enum class KindArchitecture {
    X86,
    X64
};

constexpr OperatingSystem OS = Enviroment_OS;
constexpr KindCompiler Compiler = Enviroment_Compiler;
constexpr KindArchitecture Architecture = Enviroment_Architecture;
```

여기서 모두 enum으로 변수를 만들었으며, constexpr로 상수로 선언을 하였다. 이때 대입으로 Enviroment_OS나 Enviroment_Compiler 처럼 대입을 하고 있다. 여기서 대입 하는 값들은 모두 define 으로 정의가 되어 있다.

```cpp
#if defined(__APPLE__) && !defined(Enviroment_OS)
	#define Enviroment_OS OperatingSystem::MacOS;
	#define OS_MAC true
	#define OS_WINDOWS false
	#define OS_LINUX false
	#define OS_OTHERS false
#elif (defined(_WIN64) || defined(_WIN32)) && !defined(Enviroment_OS)
	#define Enviroment_OS OperatingSystem::Windows;
	#define OS_MAC false
	#define OS_WINDOWS true
	#define OS_LINUX false
	#define OS_OTHERS false
#elif defined(__linux__) && !defined(Enviroment_OS)
	#define Enviroment_OS OperatingSystem::Linux;
	#define OS_MAC false
	#define OS_WINDOWS false
	#define OS_LINUX true
	#define OS_OTHERS false
#else
	#define Enviroment_OS OperatingSystem::Others;
	#define OS_MAC false
	#define OS_WINDOWS false
	#define OS_LINUX false
	#define OS_OTHERS true
#endif
```

OS 판별 하는것은 3가지로 기본으로 만들었으며, Apple사의 MacOS, Microsoft사의 Windows, Linux가 기본으로 인식하게 만들었다. 여기서 모두 3가지가 아닌 경우 Others로 하도록 하였다.

```cpp
// Runtime에 사용이 될 상수
#define Enviroment_OS
// define문으로 처리 할 때 쉽게 쓰기 위한 전처리문
#define OS_MAC
#define OS_WINDOWS
#define OS_LINUX
#define OS_OTHERS
```

그 외에도 Compiler도 마찬가지로 위의 사이트를 기반으로 인식하도록 하였으며 인식 범위는
MSVC, MingW, GNU재단 컴파일러만 인식하도록 하였으며 Clang이나 LLVM 같은 컴파일러 플랫폼은 인식 범위에 넣지 않았다. 그 이유는 처음 소프트웨어를 개발 할 때 컴파일러 마다 다른 결과를 내는 기능을 사용하지 않았기 때문이다.

```cpp
__attribute()__
```

# 마무리

처음 개발을 할 때 멀티플랫폼! 을 지향하면서 개발을 시작 하였지만 OS마다 글자를 출력하는 방식이나, Terminal의 커서를 움직이거나, 색깔을 출력하는 방식이 모두 달랐었다. Linux/Mac 계열은 출력을 통하여 색깔을 변경하였지만 Windows 경우에는 HDC를 이용하여 색깔을 변경하므로 OS 판별과 그에 따른 대응 코드를 달리 해야 했었다. 그래서 Core부분의 로직은 #define으로 전처리 하여 사용하였고, 추후 Core를 이용하여 개발 하는 사용자 입장에서 OS나, Compiler를 인식 해야 할 때 유용하도록 Enviroment를 만들어 제공을 하도록 하였었다.

# 개인 생각

.Net Core를 이용하여 개발을 할 때 OS의 종류나 Kernel 버전 등 다양한 정보를 제공을 해주었었는데 .Net Core 설치를 할 때 미리 메타데이터를 만들어서 제공을 해주는 방식 또한 존재가 가능 하겠구나 생각이 되었다. 현재 개발을 생각하고 있는게 Symless 사의 Synergy 프로그램이긴 하나, Synergy 프로그램이 Network 방식으로 동작하고 있으므로 Network 방식을 바꿔 Serial 통신으로 바꿔보고 싶다. 그러기 위해선 Apple사의 Library를 사용하고, MS사의 Library등, Linux의 Quartz 등 다양한 라이브러리를 OS 판별해서 개발을 해야 한다.

# 전체 코드

```cpp
#ifdef __ENVIROMENT_H__
#define __ENVIROMENT_H__

// OS
#if defined(__APPLE__) && !defined(Enviroment_OS)
	#define Enviroment_OS OperatingSystem::MacOS;
	#define OS_MAC true
	#define OS_WINDOWS false
	#define OS_LINUX false
	#define OS_OTHERS false
#elif (defined(_WIN64) || defined(_WIN32)) && !defined(Enviroment_OS)
	#define Enviroment_OS OperatingSystem::Windows;
	#define OS_MAC false
	#define OS_WINDOWS true
	#define OS_LINUX false
	#define OS_OTHERS false
#elif defined(__linux__) && !defined(Enviroment_OS)
	#define Enviroment_OS OperatingSystem::Linux;
	#define OS_MAC false
	#define OS_WINDOWS false
	#define OS_LINUX true
	#define OS_OTHERS false
#else
	#define Enviroment_OS OperatingSystem::Others;
	#define OS_MAC false
	#define OS_WINDOWS false
	#define OS_LINUX false
	#define OS_OTHERS true
#endif

// Compiler
#if defined(_MSC_VER) && !defined(Enviroment_Compiler)
	#define Enviroment_Compiler KindCompiler::MSVC;
	#define COMPILER_MSVC true
	#define COMPILER_GNUC false
	#define COMPILER_MINGW false
	#define COMPILER_OTHERS false
#elif defined(__GNUC__) && !defined(Enviroment_Compiler)
	#define Enviroment_Compiler KindCompiler::GNUC;
	#define COMPILER_MSVC false
	#define COMPILER_GNUC true
	#define COMPILER_MINGW false
	#define COMPILER_OTHERS false
#elif (defined(__MINGW32__) || defined(__MINGW64__)) && !defined(Enviroment_Compiler)
	#define Enviroment_OS KindCompiler::MingW;
	#define COMPILER_MSVC false
	#define COMPILER_GNUC false
	#define COMPILER_MINGW true
	#define COMPILER_OTHERS false
#else
	#define Enviroment_OS KindCompiler::Others;
	#define COMPILER_MSVC false
	#define COMPILER_GNUC false
	#define COMPILER_MINGW false
	#define COMPILER_OTHERS true
#endif

// Architecture
#if OS_WINDOWS
	#if defined(_WIN64)
		#define Enviroment_Architecture KindArchitecture::X64
		#define ARCHITECTURE_X64 true
		#define ARCHITECTURE_X86 false
	#elif defined(_WIN32)
		#define Enviroment_Architecture KindArchitecture::X86
		#define ARCHITECTURE_X64 false
		#define ARCHITECTURE_X86 true
	#endif
#elif OS_LINUX
	#define Enviroment_Architecture KindArchitecture::X86
	#define ARCHITECTURE_X64 false
	#define ARCHITECTURE_X86 true
#elif OS_MAC
	#define Enviroment_Architecture KindArchitecture::X86
	#define ARCHITECTURE_X64 false
	#define ARCHITECTURE_X86 true
#else
	#define Enviroment_Architecture KindArchitecture::X86
	#define ARCHITECTURE_X64 false
	#define ARCHITECTURE_X86 true
#endif

namespace Enviroment {
    enum class OperatingSystem {
        Windows,
        Linux,
        MacOS,
        Others
    };
    enum class KindCompiler {
        MSVC,
        MingW,
        GNUC,
        Others
    };

    enum class KindArchitecture {
        X86,
        X64
    };

    constexpr OperatingSystem OS = Enviroment_OS;
    constexpr KindCompiler Compiler = Enviroment_Compiler;
    constexpr KindArchitecture Architecture = Enviroment_Architecture;
}
#endif
```
