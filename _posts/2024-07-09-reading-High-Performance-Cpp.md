---
layout: post
title: 고성능을 위한 언어 C++ 책을 읽고나서
author: piorosen
categories: [Blogging, Review]
tags: [read,cpp,book,high,performance,high-performance]
hide_title: false
---

# 개요

이번에 읽은 책으로는 기존에 공부하였던 내용들에 대해서 한번 더 복습하는 기회가 되었다. 그리고 C++ 20 에 대해서 한 번 공부 할 수 있는 기회였다. 아무래도, C++ 20이나 C++ 23은 책으로된 내용이 많이 부족하다 보니 좋은 기회였다. 그렇지만 다음에 이 책을 읽을 기회가 있는 새로운 독자의 경우에는 충분한 C++ 지식이나 다른 언어에 대한 선행 지식이 있어야 읽기 편할것 같다.

무언가 C++에 대해서 깊고 심오함을 공부하고 싶다면 `Optimized C++`를 읽어보는것이 더 좋을 것이며, 이 책의 경우에는 C++ 17과 C++ 20에서 메모리 관리와 새로운 문법에 대해서 공부에 집중한 느낌이였다.

책의 페이지가 총 450p 정도 되는 양이였고, 하루에 100p 정도 읽어서 4일만에 다 읽었기 때문에, 충분한 사전 지식이 있다면 술술 읽을 수 있을것이라 생각한다.

# 핵심 내용

필자가 재밌게 읽었던 부분을 요약하자면 아래와 같고, 이름은 필자가 조금 변형하였다.

1. 캡처가 된다면 람다는 클래스화가 되는거였다! p76
2. 이동 시멘틱은 나 천재인듯! 이제야 이해하다니!! ㅠㅠ rust의 소유권 이전과 동일하구나! p100
3. 평행 배열이 존재할 때, 구조체의 변수의 크기가 작다면, 캐시 라인에 많은 배열이 들어가므로, 속도 개선이 된다. p158
4. `Iterator` 직접 구현해서 `LinearIterator` 만들어보기 p180
5. `swap`이나 `std::move`를 사용하더라도, `noexcept`로 명시해야함. p189
6. `std::ranges::view` 에는 다양한 Lazy Evaul or functional 기능이 있다 C++20, p206
7. `std::ranges::action` 즉각적인 평가를 수행하고, `Mutable` 하다.
8. C++ 자원관리 `RAII (Resource Acquisition is Initialization)` p239
9. 명시적 새로운 new 선언! 신기했다. p243
10. `Reflection`을 `tie`로 구현하는 재미 310p
11. `shared_ptr` 복사와 생성은 쓰레드 안전하지만, 안에 있는 데이터는 쓰레드 세이프 하지 않다. p394 + cppreference

# 요약

1. 캡처가 된다면 람다는 클래스화가 되는거였다! p76

이것은 이전부터 설명하던 내용이여서 바로 생략을 하도록 한다.

2. 이동 시맨틱은 rust의 소유권 이전과 동일하다.

`std::move`나 `std::forward`에서 이 함수들은 도대체 무슨 역할을 하는지 이해하게 되었다. 정말 단순하게 생성자와 복사 행위를 멈추기 위해서, `auto p = std::move(var)`를 사용하여 `var`에 대한 접근 권한을 빼앗고, `p`에게 전달하는 것이다. 이것이 왜 필요한가에 대해서는 함수로 넘어가게 된다면 말이 달라지게 되는 것이였다.

극히 정말로 단순하게 함수에서 구조체나 클래스가 넘어가게 될 경우 포인터나 레퍼런스 타입으로 값을 전달하지 않는다면 반드시 생성자나 복사가 발생하게 된다. 하지만 가끔 사람들이 포인터나 레퍼런스 값을 전달해준 이후에 소유권 자체를 완전히 넘겨주고 싶은 경우가 존재한다. 예시로 `std::unique_ptr`이 있으며, 단일 객체로만 존재해야하는 경우도 존재한다. 그렇기 때문에, `std::unique_ptr`의 경우에 복사나 생성자가 호출되는 순간 2개 이상의 객체가 생성이 되므로 문제가 생기게 되고, 포인터로 넘겨지게 된다면 해당 객체를 가르키는 변수가 2개 이상이므로 문제가 생기게 된다.

그래서 이러한 문제를 없애기 위해 `std::move`와 `std::forward`가 존재하게 되는 것 이였다. 크게 어렵게 생각할 이유가 없는것 같다.

3. 평행 배열이 존재할 때, 구조체의 변수의 크기가 작다면, 캐시 라인에 많은 배열이 들어가므로, 속도 개선이 된다. p158

이 부분에 대한 내용은 정말로 신기하였으며, 당연하면서도 새로운 기분을 주는 듯한 내용이였다. 구조체의 데이터가 배열 형태로 존재하고, 구조체의 크기가 작으면 작을 수록 한번의 캐시로의 데이터 이동으로 많은 데이터를 옮길 수 있게 되므로 성능 향상에 도움이 되는 것이다.

그렇다면 기존에 거대해진 구조체를 어떻게 성능 향상을 시킬수 있는가? 에 대한 질문으로는 포인터를 적극 활용하여 해결 할 수 있다는 것이다. 즉, 구조체 1개로 모든 데이터를 담는 것이 아닌, 구조체 3개 4개로 분할해서 만들고, 1개의 구조체에 나머지 구조체를 포인터 형태로 간접 접근 하는 방식으로 채택한다면, 하나의 캐시 라인에 최대한 많은 데이터가 모두 들어갈 수 있다. 

단점으로는, 포인터화되어서 간접적으로 접근하게 된 값은 2중 참조하여야 접근이 되므로 더 많은 성능을 요구하게 된다라는 것이 큰 단점이다. 그렇기 때문에 적절한 조절이 필요한, 그리고 최적화가 가능한 부분이라고 필자는 생각한다.

8. C++ 자원관리 `RAII (Resource Acquisition is Initialization)` p239

정말로 몰랐던 내용으로는 `Stack`과 `Heap` 메모리에 대한 정의나 실질적으로 활용되는 언어는 극히 드물다는 것을 알게 되었습니다. 대표적으로 `Java`나 `Python`의 경우에는 모든 변수들을 `Heap`에만 저장하므로 `Stack`에서는 따로 관리가 되고 있지 않다는 것을 알게 되었습니다. 물론 `Java`의 경우에는 모든 변수가 포인터 형태로 관리가 되고 있기 때문에, `Stack`을 활용하기도 하나, 대부분은 생성되는 변수는 `Stack`에는 포인터 변수가, `Heap`에는 실제 데이터가 있는 구조로 되어 있습니다.

그래서 `Stack`에 데이터 저장과 `Heap`에 데이터를 저장을 원하는대로, 그리고 명시적으로 지정이 가능하므로 이를 활용한 자원 관리 기법이란 것이 나타났는데, 그것이 바로 `RAII(Resource Acquisition is Initialization)`입니다.

`GC(Garbage Collect)` 없는 C++ 에서는 `Stack` 메모리에서 데이터의 삭제를 시기를 기준으로 자원을 관리하는 것 입니다. 즉, 아래와 같은 코드로 자원을 관리 할 수 있습니다.

```cpp
//* 어딘가 Timer란 구현체가 존재함.

{
    ScopedTimer t("Something Work!");

    // Do Working
}
```

위의 코드를 실행하게 되었을 때, `t`란 변수가 `Stack`으로 부터 파괴가 되었을 때 실행 시간을 출력하는 기능을 만들 수 있습니다.

11. `shared_ptr` 복사와 생성은 쓰레드 안전하지만, 안에 있는 데이터는 쓰레드 세이프 하지 않다.

뭔가 처음에는 `shared_ptr`에 있는 `counter`가 `atomic`하다고 알고 있었는데, 이 말이 무슨말인지 처음에는 이해가 잘 되지 않았다. 하지만 다시 생각해보면 당연하다는 말이라는 것을 알게 되었다.

`shared_ptr`이 복사나 대입을 통해 새롭게 생성이 된다면 `counter`가 `atomic`하므로 멀티 쓰레드 환경에서 문제가 발생하지 않는다. 하지만 멀티 쓰레드 환경에서 `shared_ptr`을 사용한다면 말이 달라지게 된다 `shared_ptr`에 있는 데이터는 `atomic`한 것이 아니기 때문이다. 즉, `shared_ptr`이 가르키는 포인터는 `atomic`하지 않고, 오직 `counter`만이 `atomic`하다.

원문으로는 아래와 같으며, [[공식 문서]](https://en.cppreference.com/w/cpp/memory/shared_ptr) 이다.
```
All member functions (including copy constructor and copy assignment) can be called by multiple threads on different shared_ptr objects without additional synchronization even if these objects are copies and share ownership of the same object. If multiple threads of execution access the same shared_ptr object without synchronization and any of those accesses uses a non-const member function of shared_ptr then a data race will occur; the std::atomic<shared_ptr> can be used to prevent the data race.
```

정말로 당연하면서도,,, 실수를 많이 할 것 같은 내용이였다.


# 결론

일부 설명하지 않은 내용인 C++ 20에 대한 내용(`views`나 `action`), `Memory Arena`와 같은 추가적인 메모리 생성/파괴 관련 최적화(`new`, `delete`) 내용 또한 워낙에 쉽게 설명이 되어 있어서 정말로 좋았다.

C++ 20에 대한 내용을 요약에 담지 않은 이유로는 한번 직접 읽어 보았으면 하는 느낌이 강하였으며, `Memory Arena`의 경우에는 필자가 아직도 완벽하게 이해를 못한 것 같으므로, 내용을 생략하게 되었다.

필자가 이해한 `Memory Arena`를 간단하게 설명하자면 거대한 메모리를 사전에 생성하고 new 나 delete 직접 구현하여 Memory Allocator를 제공한다. 그리고 사전에 생성된 거대한 메모리가 부족하다면 추가적으로 `Heap`에 메모리를 할당하는 방식으로, 그냥 새로운 메모리 `Allocator`를 위한 공간을 생성하는 객체라고 생각하게 되었다.

그렇지만... 필자가 간단하게 검색한 `Arena`는 그러한 것이 아니였으므로 조금 더 근본적인 `Memory Arena`에 대해서 공부할 예정이다.
