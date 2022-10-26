---
layout: post
title: 오목 게임 개발을 위해서 CMake와 OpenGL를 활용한 프로젝트 개발
author: piorosen
tags: [cmake, opengl, stb, makefile, mingw, gcc, spdlog, rapidjson, glut, thirdparty]
categories: [Blogging, Develop]
hide_title: false
# feature-img: "assets/img/feature-img/"
# feature-img: "http://placehold.it/700x300"
# thumbnail: "assets/img/thumbnails/feature-img/"
# thumbnail: "http://placehold.it/200x200"
---

# 왜 오목 게임을 개발 하게 되었는가?

교수님의 요청으로 올해에 고등학생 R&E 활동을 보조하게 되었다. 주제는 "오목 인공지능 및 학습 프로그램 개발" 이였다. 교수님과 몇번의 상의와 AI를 어떻게 작성할 것인지, GUI는 어떤 방식으로 구현 할 지 내용을 많이 나눴다.<br>
고등학생이 C++와 Python만 공부한 상태였기에, 교수님이 C++이나 Python으로 프로그램을 짜는것을 원하셨다. Python으로는 수백줄을 짜기에는 너무 힘들었고, 인텔리센스에 대한 불만이 많았었기에, C++으로 개발하게 되었다. C++ 언어로 무언가를 만든다는 것은 처음이였기에 옛날에 접했던 지식 활용이 많이 필요 했었다. 대표적으로는 vcpkg나 cmake, nuget과 같은 패키지 매니저 부터 시작해서, 빌드 시스템, 그리고 개발을 진행할 IDE 까지 생각이 많이 필요 했었다. <br>

# 왜 CMake와 OpenGL을 사용하게 되었는가?

아무래도 [과거에 사용했던 CMake](/2021/06/18/cmakeosdefine.html)때문인지 모르겠지만 cmake나 ninja와 conan, buckaroo 보다는 익숙 하고 전에 공부 했던 cmake를 사용 하게 되었다. 나중에 천천히 다른 빌드 시스템 또한 공부해볼 예정이다. 개발 IDE 영향이 있었는데 목표로 하는 타겟 OS가 윈도우와 맥이였다. 그렇기 때문에 Visual Studio나 XCode가 아닌 다른 IDE를 사용해야 했었고, 다른 대안책으로 Visual Studio Code를 택하였다.<br>
그리고 C++ 을 이용해서 GUI 개발을 할 때 다양한 라이브러리, 프레임워크를 찾아 보았다. 대표적으로는
- OpenGL (GLUT)
- QT6
- Nana
- GTK+
이 있었는데 교수님이 나중에 추후 자신이 수정을 해야할 때 편한 OpenGL이나 QT로 하자고 하셨다. 그래서 QT와 GLUT 둘 중 선택을 해야 했었다. 그러나 QT는 취업 하고 난 뒤에 실제로 사용할 일이 없을 것 같아 QT를 배제하고 OpenGL을 선택하게 되었다. (실제로는 OpenGL 또한 엄청 구버전인 2.0으로 개발 하기 때문에 실제로 사용할 일은 없을 것 같다.)

# C++을 개발 하면서 생긴 문제 점

아무래도 가장 큰 문제로는 다른 프로그램의 라이브러리를 어떻게 들고 올 것인가? 란 주제가 제일 큰 문제였다. 왜냐하면 Visual Studio라면 vcpkg란 패키지 매니저가 따로 존재 했고, XCode 또한 빌드 매니저가 따로 존재 했었기 때문이다.
CMake로 다른 라이브러리를 가져오는것은 어떻게 가져오는지 알아 보다가 ExternalProject_Add 라는 명령어가 눈에 들었다. 그 덕분에 C++ 개발 하기 위한 라이브러리를 "git clone" 하고, 헤더만 포함 한 뒤 헤더를 불러와서 사용 하였다.
처음에는 target_include_directory를 어떻게 사용 해야하는지, CMakeLists.txt 파일을 자주 보았는데 *.cmake 파일은 또 무엇인지 많은 공부가 필요 했었다. "*.cmake"는 C++ 언어로 표현하자면 헤더 파일 같은 친구이다. include() 해서 안에 있는 명령어를 호출 할 수 있으며, 파일을 분할 처리하는 느낌이다.

# 개발 하면서 다양한 문제를 직면, 해결 방법
### 1. OpenGL 에서는 이미지 관련 라이브러리가 없다!

놀랍게도 OpenGL에서는 이미지 관련된 라이브러리가 전혀 없다. 이해가 가지는 않지만 문서로서는 OpenGL 자체로는 vertex를 입력 받은 뒤 그래픽 카드에 보내 화면으로 출력하는 역할만 한다. 그렇다면 우리가 흔히 보던 이미지, 텍스처, 질감, 쉐이더 등 다양한 처리는 어떻게 어디서 하는지 의문이 많이 들 수 있다. 그러한 내용은 glut나 glew와 같은 추가 확장 라이브러리를 사용 해야 한다. 
- glut : Window의 키보드, 마우스, 팝업창 같은 OS에 따른 입력을 처리 함.
- glew : Image 프로세싱, 텍스처, 질감, 쉐이더 관련 처리를 함.

여기서 이미지를 렌더링 하기 위해서 glew를 사용 하려 했었다. 그렇지만 glew에서만 사용하는 데이터 포맷이 존재 하면서 glew와 호환이 되는 이미지 프로세싱 라이브러리가 따로 필요 했다. 흔히 윈도우에서는 HWND 에서 Window 핸들을 가져와 OpenGL이 아닌 Windows의 기능을 활용 해서 그리는 경우가 많았다. 그렇지만 목표는 Windows와 Mac의 동시에 컴파일이 되는 멀티 플랫폼을 지향 했기 때문에 해당 기능은 사용하지 못한다.

```cpp
// texture 변수를 만들고
// glGenTextures로 새롭게 만들 texture 번호를 할당 받는다.
GLuint texture;
// 이때 1은 만들 텍스처의 개수이다.
// 6으로 하게 될 경우 texture은 6개의 텍스처를 가지게 된다.
glGenTextures(1, &texture);
// texture 변수는 GL_TEXTURE_2D 형식이고, 이제 작업하는 부분은 texture로 된다.
glBindTexture(GL_TEXTURE_2D, texture);

// 텍스처를 적용한다.
glTexImage2D(GL_TEXTURE_2D, 0, STBI_rgb_alpha, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, buffer);

// 텍스처의 형식이다. UV의 값이 벗어날 경우 반복 할지, 안티 앨리어싱을 어떻게 할지 정의 한다.
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);

// mipmap으로 이미지 단일화 함.
glGenerateMipmap(GL_TEXTURE_2D);
```
[참고](https://tech.burt.pe.kr/opengl/opengl-es-tutorial-for-ios/chapter-12)

그래서 glew 기능을 이용해서 개발 해야 했었지만, glew에 적용하기 위해서 이미지 파일을 읽은 뒤 메모리에 올리는 작업은 따로 지원을 하지 않았다... 그래서 
[Khronos 재단 위키](https://www.khronos.org/opengl/wiki/Image_Libraries)에서 이미지 라이브러리를 추천을 하고 있다. 그 중 sail 사용 하다가 stb를 알게 되어 stb 라이브러리를 사용하게 되었다.

RGB 성능 테스트 | RGB 성능 테스트
:---: | :---:
100*71 | 1000*709
![이미지1](https://github.com/HappySeaFox/sail/blob/master/.github/benchmarks/PNG-RGBA-100x71.png?raw=true) | ![이미지2](https://github.com/HappySeaFox/sail/raw/master/.github/benchmarks/PNG-RGBA-1000x709.png?raw=true)
6000*4256 | 15000*10640
 ![이미지4](https://github.com/HappySeaFox/sail/blob/master/.github/benchmarks/PNG-RGBA-6000x4256.png?raw=true) |  ![이미지3](https://github.com/HappySeaFox/sail/raw/master/.github/benchmarks/PNG-RGBA-15000x10640.png?raw=true)

여기서 비교해본 결과 여러 C++의 이미지 라이브러리의 성능 비교 표가 있었는데 항상 높은 성능을 가지고 있으며, OpenGL에 쉽게 이식이 되는 STB 라이브러리를 선택하고, 적용 하면서 OpenGL에 이미지(텍스처)를 적용 하였다.
(SAIL은 OpenGL로 바로 적용이 불가능 하였고, 추가적인 연산이 필요 했다.)

### 2. CMake로 다양한 라이브러리 적용.

처음 개요에서 이야기 했듯이 CMake로 여러개의 라이브러리를 적용 하는것이 처음 이였다. 그래서 어떻게 해야하는지 찾아보았으며, ExternalProject_Add에 대한 명령어를 알게 되었고, CMake에서 makefile 만들 때 git clone하게 하였다. 처음에는 ExternalProject_Add가 어떤 명령어인지 이해가 잘 되지 않았지만, 사용해보고 실제로 적용을 해보니 어떤 역할을 하는지 확실히 알게 되었다. 클론 할 때, 클론 하고 난 뒤 빌드 할 때, 환경 설정 할 때 명령어를 입력이 가능 했으며, CMakeLists.txt를 읽을 때 어떤 인자값을 넘길지 등 다양한 처리가 가능했다. <br>
처음에는 git의 기능인 submodule을 이용해서 적용을 해야하는가? 싶었지만 ExternalProject_Add 하고, target_include_directories로 헤더 파일을 추가 하도록 하였다. 물론 add_dependencies를 추가 해서 처리 순서를 지정 해 주었다.

사용한 라이브러리는 현재로는 3가지 이지만 추가적으로 더 늘어날 가능성은 존재 한다.
- [stb](https://github.com/nothings/stb) : 이미지 프로세싱
    * OpenGL로 하면서 UI부분을 만들기 위함.

- [spdlog](https://github.com/gabime/spdlog) : 콘솔 로그
    * 테스트 하면서 정상적으로 동작하는지 확인 하기 위함.

- [rapidjson](https://github.com/Tencent/rapidjson.git) : Json 라이브러리
    * AI 개발 할 때 데이터 저장, 불러오기를 구현 하기 위함.

다행이 대부분의 프로그램이 Header Only 라이브러리 이다 보니 큰 문제가 발생하지 않았다. 만약 cpp를 빌드 하고 링크 작업을 해야 했다면 ExternalProject_Add에서 빌드 명령을 내리고 링크 작업을 하는 등 다양한 추가적인 부분이 필요했을 것 이였다.

### 3. OpenGL 에서 UI 개발을 어떻게 할 것인가?

흔히 OpenGL에서는 대부분 모든 것을 직접 구현을 해야 한다. 예를 들어서 버튼을 만든다거나, 리스트뷰를 만들거나, 스크롤바, 체크박스와 같은 다양한 UI를 위한 기능은 따로 제공을 해주지 않는다. 그렇다 보니 OpenGL로 직접적으로 구현을 해줘야 했다.

> 솔직하게 이야기를 하자면 OpenGL을 활용해서 제공해주는 UI Components 라이브러리를 써도 된다는걸 알아챈건 꽤 시간이 지나고 난 후 였다.

UI를 어떻게 구현 해줘야 하는가? 라는 생각을 많이 하게 되었는데, 과거 Apple의 UIKit이나, C#의 Winform에 대해서 많은 공부를 하였기 때문에 대략적인 감을 잡고 시도 해 보았다.
큰 형식은 Apple의 UIKit과 유사하게 개발하고자 하였다.

__목표는 아래와 같았었다.__
> 내부 구조는 복잡하더라도 사용자가 사용은 간단하고 심플하게.

UIKit의 구조는 먼저 Application(Entry Point) 안에 Window(최상위 View, C#으로는 Form)가 있으며 Window 안에 ViewController(C#으로는 designer가 아닌, 코드 작성 창)가 있는 구조이다. 그래서 이 구조를 적용 했지만, 다르게 구성 하였다.

Application -> ViewController -> View 형식으로 구조를 변경 하였다. <br>
Application에서는 OpenGL에서 발생 하는 키보드, 마우스, 렌더링 함수를 받아서 처리를 하도록 하였다. <br>

Application에서는 이벤트가 발생 할 경우 root ViewController에 데이터를 전달 한다. 만약 root ViewController가 없을 경우 spdlog로 오류를 출력 한다. root ViewController에서는 view에게 이벤트를 또 다시 전파 해서 각 View에서 처리를 할 수 있도록 하였다.
이때 root ViewController를 상속 받아서 ButtonView의 이벤트를 받아서 처리를 하도록 하였다.

View가 ViewController에게 전달하는 형식으로는 delegate 패턴을 사용할지, event 패턴을 사용하지 고민 하였다. 그렇지만 당장 구현할 때 새벽이다 보니, 패턴으로 구현하자니 너무 피곤해서... View안에 대리자를 구현하거나, 이벤트 객체를 만들지 않고 바로 std::function<>형식으로 정의를 했다. 

```cpp
class view { 
    public:
    std::function<void()> keydown;
    std::function<void()> keyup;

    protected: 
    // application에서 발생한 마우스 이벤트 처리 공간
    void appMouseEvent(int state) { 
        if (state && keydown) { 
            keydown();
        }else if (!state && keyup) {
            keyup();
        }
    }
};
```

위의 형식으로 구현을 하였다.

# 그래서 현재 결과물은 어떠한가?

메인 화면 | 보드 게임 판
:---: | :---:
![이미지1](/assets/img/post/2021-09-04-main.png) | ![이미지2](/assets/img/post/2021-09-04-popup.png)
![이미지1](/assets/img/post/2021-09-04-board.png) | ![이미지2](/assets/img/post/2021-09-04-boardGame.png)

현재 상태로는 GUI 형식으로만 구현만 했다.

# 미래 목표

현재로는 GUI를 먼저 개발을 완료 하고 난 뒤에 AI를 개발 할 예정이다. 다른 사람이 오목을 둔 기보가 있어 해당 기보 기반으로 학습 할 예정이다. 
