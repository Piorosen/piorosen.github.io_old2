---
layout: post
title: UE4와 Unity에 적용된 컴포넌트 패턴 개발(CBD)을 적용 해본 사례
author: piorosen
tags: [game-engine, coding-style, components, property]
hide_title: false
categories: [Blogging, Design-Pattern]
# feature-img: "assets/img/feature-img/"
# feature-img: "http://placehold.it/700x300"
# thumbnail: "assets/img/thumbnails/feature-img/"
# thumbnail: "http://placehold.it/200x200"
---

# 개요
수 없이 많은 언어와 개발 툴, 프로그램을 만져 오면서 사용을 해보게 되면서 우연찮게 디자인 패턴이라는 분야에 대해서 알게 되었다. 그렇게 자연스럽게 소프트웨어 설계라는 학문에 처음 접하게 되면서 기존에 만들었던 소프트웨어가 얼마나 비효율적이며, 유지 보수가 힘든지, 알게 되었다. 특히 [2019년 7월 ~ 2021년 5월] 1년 10개월 동안 프로젝트를 진행 맡았던 Anti Collision System(이하 ACS)를 개발 하면서 개발한 소프트웨어가 정상적으로 동작하는지에 대한 시험 및 검증 테스트를 하기 위해서 시뮬레이터를 만들면서 디자인 패턴을 적용한 사례에 대해서 이야기를 해보고자 한다.

# 객체 지향 프로그래밍이란

객체 지향 프로그래밍(Object Oriented Programming)은 간단하게 기존 절차 지향 프로그래밍(Procedural Programming)을 조금 더 유지 보수와 여러 사람과 협업을 통해 개발을 할 수 있도록 만든 개념이다. 절차 지향 프로그래밍은 컴퓨터가 명령어를 순차적으로 처리를 한다는 개념을 활용한 내용이다. 그 와 반대로 객체 지향 프로그래밍은 "실행할 단위"를 하나의 객체로 만들어 해당 객체에서 상태와 행위에 대해서 정의 한다. 

그래서 소프트웨어를 개발 및 설계를 할 때 완전히 다른 양상을 보인다. 예를 들어 절차 지향 프로그래밍으로 게임을 만든다고 가정을 하자면 아래와 같다.
1. 게임 초기화
2. 게임 시작
3. 플레이
4. 게임 종료

이 처럼 게임을 만든다고 하면 게임의 동작이 되는 절차에 따라서 프로그램을 개발할 가능성이 높다. 하지만 객체 지향 프로그래밍으로 개발 한다고 해본다.
1. 게임 관리자 객체 
    - 게임 시작
    - 게임 종료
    - 세이브 / 로드
2. 게임 객체
    - 플레이어
        - 이동
        - 스킬
            - 액티브 / 패시브
        - 카메라
    - NPC
    - 인공지능
        - NPC 인공지능
        - 적대적 NPC 인공지능
3. 네트워크 통신 객체
    - Server
    - Client

위의 내용 처럼 개발을 파편화 해서 개발이 가능하다. 특히 오픈소스에서 가장 많이 쓰이는 부분이기도 하다.<br>
그래서 장점으로는 코드 재사용성이 매우 유용하고, API를 정의를 한다면 여러명이 동시에 개발을 하더라도 문제가 발생 가능성이 낮아진다. 
단점으로는 절차 지향 프로그래밍에 비해서 객체가 스스로의 데이터 무결성과 안정성을 가져야 하므로, 중복으로 데이터를 처리 하거나, 데이터 검사를 할 수 있으므로 절차지향에 비해서 성능이 낮다.<br>

_그렇지만... 컴퓨터가 매우 빠르기 때문에 그 정도의 최적화가 필요 하다면 어셈블리 형식으로 짜지 않을까 라는 생각을 한다._

그래서 객체 지향 프로그래밍은 유지 보수가 상당히 쉽고, 적응을 한다면 Primitive하게 객체를 만들고 관리하게 만들어 절차 지향적 보다 안전하게 개발이 가능하다.

# 디자인 패턴 이란?

[wikipedia](https://ko.wikipedia.org/wiki/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4_%EB%94%94%EC%9E%90%EC%9D%B8_%ED%8C%A8%ED%84%B4)에서 정의 하는 디자인 패턴은 아래와 같다

> 소프트웨어 디자인 패턴(software design pattern)은 소프트웨어 공학에서 소프트웨어 디자인에서 특정 문맥에서 공통적으로 발생하는 문제에 대해 재사용 가능한 해결책이다. 소스나 기계 코드로 바로 전환될수 있는 완성된 디자인은 아니며, 다른 상황에 맞게 사용될 수 있는 문제들을 해결하는데에 쓰이는 서술이나 템플릿이다. 디자인 패턴은 프로그래머가 어플리케이션이나 시스템을 디자인할 때 공통된 문제들을 해결하는데에 쓰이는 형식화 된 가장 좋은 관행이다.

디자인 패턴은 간단하게 말하여 유지 보수를 효율적으로, 코드를 재 사용이 용이하게 하거나, 비즈니스 로직을 작성하여 유닛 테스트 하기 쉽게 해준다. 그 중 나는 디자인 패턴 중 시뮬레이터를 개발 할 때 컴포넌트 패턴을 사용해서 개발 하면서 코드의 간결함과, 유지 보수에 매우 좋은 결과를 얻었었다. 그래서 컴포넌트 패턴에 대해서 설명과 실제 코드를 작성 한다면 어떠한 방식으로 짤 수 있는지 설명을 해보고자 한다.

# 컴포넌트 패턴(CBD) 이란?

게임 개발을 할 때, 또는 무언가를 개발 할 때 OOP 기반으로 개발을 주로 하는데<br>
OOP의 단점이 상속을 계속 하게 되면 내용이 비대 해지고 관리가 힘들어 지며, 거대한 코드가 되고 만다.<br>
그래서 상속(수직적) 구성이 아닌, 컴포넌트(수평적) 기반으로 새로운 패턴이 나왔다.<br>
상속을 하는것이 아닌, "컴포넌트" 단위로 개발을 하는것이고, 새로운 기능을 추가 할 때 마다 "컴포넌트"를 만들어서<br>
추가 하면 된다.<br>
<br>
실제로 동작하는 방식이나, 코드를 보면 엄청 쉽게 이해가 된다.<br>
<br>
코드 작성은 C++로 하였고, 공부용으로 사용 했다.<br>

```cpp
#include <iostream>
#include <string>
#include <list>

class component;
class object;
class userComponent;
class adminComponent;
class manageComponent;
```

제일 먼저 사용하는 헤더, 그리고 class를 정방 선언을 해주었다.

```cpp
class component {
public:
    object* attachParent = nullptr;

    virtual void print() {
    	std::cout << "component\n";
    }
};
```

먼저 component-base 라고 하였으니 component를 만들어 준다.<br>
component는 혼자서 동작을 하지 못하므로 어디에 붙는지 부모를 만들어 주고<br>
테스트 하기 위하여 print 함수를 만들었다.<br>

```cpp
class object {
private:
    std::list<component*> components;

public:
	// componet를 추가 함.
    void addComponent(component* component) {
        component->attachParent = this;
        components.push_back(component);
    }

    // componet를 가져옴. 
    // 해당 componet를 추가 하지 않았으면 nullptr를 반환함.
    template<typename T>
    T* getComponent() {
        for (auto iter : components) {
            T* item = dynamic_cast<T*>(iter);
            if (item != nullptr) {
                return item;
            }
        }
        return nullptr;
    }
};
```

그 다음에는 component를 관리하는 객체이다.<br>
해당 객체에서 component를 관리하고 지속적인 실행을 행해준다.<br>
<br>
addComponent 코드에서 component*를 받은 후 attachParent를 자기 자신으로 가르킨다.<br>
그 후 components에 추가 한다.<br>
<br>
해당 알고리즘에서는 removeComponent를 구현 하지 않았다. 추가가 가능하다면 삭제 또한 구현하기 쉬울 것이다.<br>
중요한건 addComponent와 getComponent이기 때문이다.<br>

```cpp
class userComponent : public component {
public:
    virtual void print() {
        printf("userComponent\n");
    }

    void userFunc() {
        printf("only have user\n");
    }
};

class adminComponent : public component {
public:
    virtual void print() {
        printf("adminComponent\n");
    }

    void adminFunc() {
        printf("only have admin\n");
    }
};
```

component 클래스를 상속 받는 userComponent와 adminComponent를 만들었다.<br>
userComponent에 만 구현이 되어 있는 userFunc, 마찬가지로 adminComponent에만 구현이 되어 있는 adminFunc가 있다.<br>

```cpp
int main() {
    object mainObject;

    component* user = new userComponent();
    component* admin = new adminComponent();

    mainObject.addComponent(user);
    mainObject.addComponent(admin);


    mainObject.getComponent<userComponent>()->userFunc();
    mainObject.getComponent<adminComponent>()->adminFunc();

    return 0;
}
```

그리고 main에서 각각의 컴포넌트를 만들고 mainObject에 컴포넌트를 추가 해 준 뒤,<br>
getComponent함수를 호출 하여 정상적으로 동작하는지 확인한다.<br>
<br>
only have user<br>
only have admin<br>
결과 값은 당연하게 위 처럼 나온다.<br>
<br>
자 이제, 유니티에서 사용이 되는 component안에서 다른 component를 호출을 한다면 어떻게 해야하는가?<br>

```cpp
class manageComponent : public component {
public:
    virtual void print() {
        printf("manageComponent\n");
    }

    void manageFunc() {
        attachParent->getComponent<adminComponent>()->print();
        attachParent->getComponent<userComponent>()->print();
    }
};
```

manageComponent를 구현하여 adminComponent와 userComponent를 호출 한 뒤 print 함수를 호출 하였다.<br>
<br>
구현된 방식을 보면 attachParent를 사용하여 mainObject를 가져오고, 해당 object에 있는 admin, userComponent를 가져온다.<br>
<br>
가져온 뒤 print함수를 호출 하는 부분이다.<br>
<br>
즉 코드의 흐름이<br>
mainObject -><br>
getComponent로 manageComponent를 가져옴 -><br>
mainObject를 가져 옴 -><br>
getComponent로 adminComponent를 가져옴 -><br>
print 함수를 호출 함<br>

# 그래서 어디에 사용 했는가?

우선적으로 나는 프로그램이 정상적으로 동작하는지 테스트를 위해서 시뮬레이터를 만들었다. 왜냐하면 실제 장비는 주 개발 지역인 부산이 아닌, 타 지역 대전에 있었기 때문에 매번 코드가 정상적으로 동작하는지 테스트를 하기 위해서 매번 대전으로 방문은 힘들었다. 그래서 처음 데이터 프로토콜을 받고 해당 데이터가 빅 엔디안으로 데이터가 들어오는지 리틀 엔디안으로 데이터가 들어오는지 알 방법이 전혀 없었기 때문에 처음에는 기존 장비에서 ACS로 데이터가 넘어오는 패킷만 우선적으로 읽어서 파일 형태로 저장한 뒤 집으로 데이터를 분석 한 뒤 테스트 베드를 구축 하는것을 첫번째 목표로 잡았다.

그래서 시뮬레이터를 만들게 되었고. 완성을 하게 되었다.

![이미지1](/assets/img/post/2021-08-13-2019-ue4.png)
#### <center>실제 동작 이미지</center>
<br><br>


![이미지1](/assets/img/post/2021-08-13-2019-test.png)
#### <center>과거 2019년 ~ 2020년 사이의 시뮬레이터</center>

처음에는 장비가 한정적이고, 장비의 움직임이 미리 정의가 되어 있었기 때문에 코드 상에서 장비의 각 가속도와 가속도만 정의해서 동작 하도록 만들었었다. 하지만 처음에는 해당 기능만 있어도 충분 했었지만, 장비에서 크기가 변한다거나, 가속도가 아닌 일정한 속도이거나, 장비의 이동 한계치를 지정 하거나 정말 다양한 케이스가 추가가 되면서 기존 코드를 수정하면서 유지 보수는 더 이상 불가능 하다 판단을 하게 되면서 완전히 코드를 새롭게 갈아 엎게 되었으며, 갈아 엎고 컴포넌트 기반 개발을 진행 하도록 하였다.

# 어디에 적용을 했는가?

처음에는 빠르게 개발을 해서 적용 하고자는 목표가 컸지만 이번에는 제대로 구현을 하고, 유지 보수와 나의 한계치 까지 모두 끌어 모아서 개발 해보고 싶었다. 그래서 시뮬레이터 코드와 GUI 코드를 완전히 분리해서 설계를 하였다.<br>
그래서 GUI와 시뮬레이터는 완전히 별개의 프로젝트로 구성이 되어있다. 그래서 시뮬레이터는 터미널에서도 동작이 가능하며, GUI 도 동작이 가능하다. 다른 개발자가 마음을 먹는다면 다른 UI도 만들 수 있도록 하였다. <br>
시뮬레이터에서 장비와 장비의 움직임에 대해서 컴포넌트 형식으로 작성을 하였다. 컴포넌트 형식으로 정의를 하게 되면서 데이터를 배열이나 반복문으로 구조를 표현이 가능하게 되면서 시뮬레이터 구조를 json으로 저장하여 로드 및 세이브 기능을 쉽게 추가 하게 되었다. 

![이미지1](/assets/img/post/2021-08-13-2021-config.png)
#### <center>실제 2021년 최종본 설정 파일</center>

컴포넌트 형식으로 작성하게 되면서 기존에 비해서 코드의 양은 약 30% 정도 줄었으며, 실린더의 움직임을 정의 하거나, 장비간 상호 움직임, 상대 위치, 장비의 회전, 실린더의 움직임에 의한 장비의 회전, 직선 운동 등 다양한 정의를 컴포넌트 형식으로 미리 정의를 하고 설정에서 모든 관계에 대해서 정의 하면 동작하도록 개발을 하였다. 그로 인해 프로그램을 수정을 하거나, 유지 보수를 해야 할 일이 줄어들게 되었다.

# 적용 전과 적용 후의 차이

컴포넌트 형식 이전에 작성 했던 내용은 분명히 빠르게 개발을 하고, 빠른 결과를 낼 때는 분명히 좋은 장점이 있다. 하지만 단점으로는 새로운 기능이 추가가 되거나, 코드가 변경이 되어야 한다면 유지 보수가 매우 어려웠다. 그래서 새롭게 소프트웨어 설계부터 어떠한 디자인 패턴을 사용해서 어떻게 해서 잦은 유지 보수를 견딜 수 있는 프로그램이 될 수 있을지 고민을 해서 만든 프로그램은 완전히 새로운 기능이 추가가 되지 않는 한 모든 내용은 설정 파일로 모든게 해결이 가능했었다. 

설정 파일만으로 해결이 된다는 것은 유지 보수는 더 이상 개발자의 영역이 아닌, 소프트웨어를 사용하는 사용자에게 있으므로 조금.. 많이 편하고 좋았다.<br>
(특히. 개발 컴퓨터가 아닌 다른 컴퓨터에서 실행만 가능한 상황이고, 프로그램 수정이 일어나야 할 때 매우 효율적이고 너무 좋았다...)

# 디자인 패턴.. 그래서 정말로 좋았는가?

처음 시뮬레이터를 작성 할 때 최대한 많은 디자인 패턴에 대해서 공부하고, 적용을 하고자 했었다. 그렇지만 억지로 디자인 패턴을 끼운다는 느낌도 없지 않아 있다. 
예를 들어서 적용 했던 패턴은 아래와 같다.
1. 컴포넌트 패턴
2. 빌더 패턴
3. 오브젝트 풀
4. 추상 팩토리 패턴
5. 싱글톤 패턴
6. 어댑터 패턴
7. 책임 연쇄 패턴
8. 브리지 패턴
9. 퍼사드 패턴
10. 프록시 패턴

여기서 분명히 더 많은 패턴을 적용 했지만 용어를 정확히 몰라서 안 넣은 부분이 있을 거라고 생각을 한다. <br>
처음에는 디자인 패턴을 적용 한다고 해서 정말 좋은 것! 이라 해서 적용을 했지만 막상 잘못 적용을 해서 코드를 전부 지우고 다시 작성을 했었다. 몇번을 코드를 지우고 다시 작성을 해보면서 디자인 패턴은 결국 하나의 코드를 작성하는 코드 스니펫과 같은 부분이라고 느낌을 많이 받았다.

그래서 느낀 부분은 처음 사용 해보는 디자인 패턴은 최대한 자제를 하고, 코드를 작성 할 때 디자인 패턴을 적용 하더라도 반드시 여기에 넣으면 좋을까? 라는 생각을 해본 뒤에 적용을 해야 한다는 생각을 많이 받았다. 그러니까 맹목적인 디자인 패턴을 찬양을 하는것 보다 한번 의심을 해본 뒤에 적용을 해보자는 생각이다.

# 레퍼런스

[관련 논문](https://www.dbpia.co.kr/journal/articleDetail?nodeId=NODE09349359) 프로그램을 작업하고 2019년과 2020년 사이에 진행 했던 개발 내용에 대한 논문이다.<br>
개발은 시뮬레이터만 아닌 충돌 검사, 로그 시스템, 리플레이 기능 등 다양한 기능이 함께 개발이 되었다. (너무.. 양이 많았지...)<br>
![이미지1](/assets/img/post/2021-08-13-all-system.png)
#### <center>최종 전체 시스템</center>