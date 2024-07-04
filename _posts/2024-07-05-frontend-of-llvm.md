---
layout: post
title: Programming Language with LLVM in Udemy 인강을 듣고 나서
author: piorosen
categories: [Blogging, Review]
tags: [udemy, llvm, lagnauge, online, course]
hide_title: false
---

# 개요

본 글은 정말로 뻘글이 될 수 도 있으며, 단순히 LLVm의 프론트엔드를 어떻게 만들어야 하는지, LLVM에 대해서 조금 더 공부해보기 위해서 인강을 수강하게 되었다.
특히, 나만의 컴파일러를 만들어 보고싶다는 생각은 코딩을 시작할 때 부터 가졌었던 나만의 욕심중 하나 였던것이 조금 컸었던것 같기도 하다.

해당 프로젝트는 필자가 중학교 2학년 때, 처음으로 한번 인터프리터 언어를 만들어 보고 싶어서 만들게 된 프로젝트이다. [[Korea-Compiler]](https://github.com/Piorosen/Korea-Compiler)는 정말로 단순한 구조를 가졌으며, 어떻게 보면 중학교 2학년 학생이 만들만한 문법 구조이다.

이 처럼, 필자는 약간 컴파일러나 인터프리터를 만들어 보고 싶었던 것 도 있었으며, 대학원 진학을 컴파일러 분야로 가게된 것도 하나의 인강을 듣게된 이유가 되었다.
솔직하게는, 책이나 자료가 너무 없었기도 했고 빠르게 LLVM 프론트엔드가 어떻게 구성이 되었는지 궁금하였기 때문에 인강을 구매하게 된 이유이다.

# 내용

내용 자체가 후기다 보니, 강의에 대한 내용 및 코드에 대해서 직접적으로 언급은 하지 않는다.

만약 강의에 대한 `Demo`에 대한 내용은 [[유튜브]](https://www.youtube.com/watch?v=Lvc8qx8ukOI) 에서 먼저 들을 수 있다. 그리고 내용 자체가 `중급자`로 잡혀져 있기도 하며, 사전에 충분한 컴파일러와 C++에 대해서 알고 있어야하는 내용이 많다.

내용으로는 `Eva`라는 새로운 언어를 만들고, 해당 언어에 대해서 `Parser`에 대한 구현은 생략하고, 오직 `AST`와 `Code Generation` 분야에 대해서만 서술되어지고 있다. 
이때, 최대한 `Eva`라는 언어는 `LLVM IR`을 생성하고, `clang++`을 통해서 실행 바이너리를 생성하는 구조이다. 때문에, 강의는 오직 `Frontend`, `AST`와 `Code Generation` 을 수행한다. 

공부를 하긴 했지만,,, 너무 추상적이기도 하며 필자가 이해를 잘 못한것 같기도 하다. 불구하고,,, 배운 내용을 기억하고자 정리를 하자면 아래와 같다.

1. `dynamic library`를 `IR` 생성하는 단계에서 `linking` 및 `Verify` 할 수 있다.
- `printf` 나 `malloc` 을 외부에서 가져올 수 있다.
- `Stack`에 메모리 할당 하는 것은 `alloca`이라는 `LLVM IR`의 `ISA`에서 정의할 수 있지만 `Heap`에 정의하려면 `malloc`을 사용하더라~

2. 결국엔 코드 생성하기 위해서 변수 선언 부터 모든것을 만들어야 한다.
- 변수를 만들고 싶으면 Create Variable 해서 생성해야하고, 임시 변수를 사용하거나 무엇을 하던지 간에 생성해야한다.

3. 변수나 함수를 생성했다면, 컴파일러 내부에서 기억하고 있어야 한다.
- `LLVM`은 변수나 함수를 생성하였다면 IR 생성하고 끝나지, 직접적인 관리는 `Frontend`나 `Own Compiler` 에서 관리를 해야한다.
- 하지 않았다면 잃어버린 것이다.

4. `Control Flow`는 대박이다! 모든 것이 `GoTO`로 이뤄진다. 그것은 `GOTO`이다 
- `if` 문으로 이뤄졌다면 `goto`는 2개로 이뤄져 있다. 
    - `if` 문 내부와 `if` 문이 끝났을 때

5. 구조체는 `GEP` 이라는 LLVM의 명령어로 동작된다.
- `LLVM IR`에서는 구조체를 생성하고 나서 각 내부 변수마다 인덱스를 지정한다.
- 해당 인덱스는 추후, `GEP` 이라는 `Inst`에서 이름이 아닌, 인덱스로 호출된다.
- `GEP`은 다용도로 사용이 가능하도록 만들기 위해서 `배열 구조체 구조체` 라면 Index 3개(varadic) 지정 할 수 있도록 되어 있다.

6. 비 캡처 함수와 람다 함수는 구현적으로 `매우 매우` 다르다.
- 비 캡처 함수는 컴파일러 입장에서 함수와 동일하므로, 이름만 랜덤한 난수로 지정하면 `Lambda-Lifting` 기법을 통해 만들어 질 수 있다.
- 캡처 함수는 생각해보자! 캡처된 변수를 `Heap` 저장하고 `Free` 할 것인가? 너무 비효율적이지 않는가?
- 그렇다면 `전역 변수(Global)`하게 만들 것인가? 그렇다면 만약, Closure 처럼 함수를 생성하는 함수 구조라면 어떻게 될 것인가? 매번 생성하기에는 어렵지 않는가?
- 그래서 람다 함수는 클래스화하여, 함수 포인터와 캡처된 변수를 함께 소유하고 있는 구조라면 가능하다!
- 그렇다! 본 강의에서는 함수형 프로그래밍의 `Closure`는 호출 가능한 객체와 동일하다고 서술되어지고 있다. 

7. 클래스 상속에서 발생 가능한 가상 함수는 정말로 신기한 구조였다.
- 필자가 작성한 2가지의 예시를 통해 가상 함수에 대해서 완벽! 이해를 할 수 있을거라 생각한다.

필자가 작성한, 클래스에 대한 예시 코드이다.

```cpp
using FuncPtr = void(*)();
class Base {
public:
    virtual void show() {
        std::cout << "Base class" << std::endl;
}};

class Derived : public Base {
public:
    int data = 0;
    void show() override {
        std::cout << "Derived class " << data << std::endl;
}};
```

먼저 `Base` 클래스와 `Derived` 클래스의 크기를 확인해보자. 아래의 값을 확인해 본다면, 빌드 환경이 `x64`라면 `8`, `16` 이 나타나게 되고, `x86`이라면 `4`, `8`이 나타나게 된다.
즉, 이를 통해 무언가가 모르겠지만 `virtual` 이란 것 때 포인터 변수가 하나 존재한 다는 것을 실험적으로 확인 할 수 있다. 그리고 해당 변수가 사실 `vtable`, 가상 테이블(함수 목록을 간접적으로 가르키는) 변수라는 것을 실험적으로 확인해 볼 수 있다.

```cpp
int main() {
    cout << sizeof(Base) << " " << sizeof(Derived) << "\n";
    return 0;
}
```

아래의 코드에서 강제적으로 캐스팅 연산을 수행하고, 함수 포인터로 호출하는 과정을 나타내보았다. `Derived`에도 `vtable`이 존재하므로 `vtable`에 있는 첫번째 함수 포인터가 `show` 이므로 `show in Derived`가 호출이 되는 것을 알 수 있다. 

```cpp
int main() {
    Derived d;
    void** vptr = *reinterpret_cast<void***>(&d);
    FuncPtr func = reinterpret_cast<FuncPtr>(vptr[0]);
    func();
    return 0;
}
```

만약에 `vtable`이 잘 이해가 되지 않는다면 아래의 그림을 보면 이해가 된다. (정말 뇌 뺴고 ChatGPT에게 맡기니 잘 그려준다...)

```
Base Class:
  +---------------------+
  | vtable pointer -----|----+
  +---------------------+    |
  | VirtualFunction1()  |    |
  +---------------------+    |
  | VirtualFunction2()  |    |
  +---------------------+    |
                             |
                             v
                     +---------------+
vtable for Base      | &VirtualFunc1 |
Class:               +---------------+
                     | &VirtualFunc2 |
                     +---------------+

Derived Class:
  +---------------------+
  | vtable pointer -----|----+
  +---------------------+    |
  | VirtualFunction1()  |    |
  +---------------------+    |
  | VirtualFunction2()  |    |
  +---------------------+    |
                             |
                             v
                     +---------------+
vtable for Derived   | &DerivedFunc1 | (overrides VirtualFunc1)
Class:               +---------------+
                     | &VirtualFunc2 | (inherits from BaseClass)
                     +---------------+
```


# 후기

결론적으로는 아직까진 자세히 (완벽하게는) 모르겠지만, 어셈블리와 같이 모든 코드를 작성해줘야한다는 것은 알게 되었다. 또한, 클래스 부분에서 필자가 알던 것 처럼 `Class`는 사실 구조체에 `정적 함수`로 바뀐다는 것을 알고 있었지만 이를 강의에서 직접 코드로 보여줌으로 써 얻는 희열도 있었다.

그리고 가장 신기하였던 부분으로는 상속으로 인한 `vtable`이란 내용에서 이전에 읽은 `Optimized C++` 책에서 `가상 함수`는 매우 느리다 란 내용에서 왜 느린지, 어떻게 동작이 되는지에 대해서 컴파일러 레벨에서 공부 할 수 있어서 매우 좋은 기회가 되었다.

[![](/assets/img/post/2024-07-05-01.jpg)](https://www.udemy.com/certificate/UC-b8d5cd33-2c1a-4717-83b3-b0ef241bc2d2/)
