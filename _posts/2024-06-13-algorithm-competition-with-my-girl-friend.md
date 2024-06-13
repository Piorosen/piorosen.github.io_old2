---
layout: post
title: 프로젝트, 누가 더 빠르게 알고리즘 문제를 푸는가?
author: piorosen
categories: [Blogging, Develop]
tags: [github-action,github,algorithm,automatically]
hide_title: false
---

# 개요

너와 나! 경쟁이다! [[프로젝트 코드]](https://github.com/oMFDOo/whimpering.git) [Issue on Github](https://github.com/oMFDOo/whimpering/issues) 에서 문제 번호와 제목, 번호를 입력하고 대결을 신청한다면 자동적으로 대결이 성립이 된다. 그리고 가장 먼저 정답을 푼 사람이 `프로그램 코드`를 작성하면 승리자가 결정이 되고, 참가자의 모두가 `프로그램 코드`를 작성하면 자동적으로 `Issue`가 `Close`되고 게임은 끝나게 된다.

Score 현황 판|게임 진행판
:---:|:---:
![](/assets/img/post/2024-06-13-01.png)|![](/assets/img/post/2024-06-13-02.png)

#  구조 설계

먼저 `1 vs 1` 구조이면서, 여러 변수를 고려하기 귀찮음으로 인해 게임 진행하는 플레이어의 값을 매직 넘버로 지정하거나, Github Repo의 값을 매직 넘버로 지정하였음을 먼저 밝힌다. 그러므로 코드를 수정이 필요하다면 상당히 많은... 귀찮음이 동반할 것이라 예상이 된다.

가장 큰 구조는 3개의 파트로 나눠져있다.

1. 어떻게 게임을 진행할 것인가?
2. 점수를 시각적으로 나타내는 현황판
3. 자동적으로 어떻게 코드를 저장할 것인가?

## 어떻게 게임을 진행할 것인가?

먼저 게임의 진행하는 과정에서 알고리즘 푸는 사이트는 정말로 많고 다양하다. 예시로 국내의 경우 백준, 프로그래머스가 있기도 하며, 해외로 넘어가게 Leet Code나 CodeForce등 더 다양하다. 그러므로 자동적으로 저지 사이트의 정보를 수집하여 Github에 정보를 수정하는 것은 매우 어렵고(시간과 노동에 비해서) 복잡하다. 그래서 저지 사이트의 정보를 수집 하는 것이 아닌, 사용자가 직접 코드를 Github Issue로 등록하게끔 구조를 설계하였다.

코드를 Github Issue로 등록하게끔 만들었다면 이제 대부분의 문제는 해결이 된 것과 같다. 그 이후 부터는 하나의 규약과 같이 하나씩 하나씩 정의하기만 한다면 무엇이든지 OK가 될 수 있다. 그래서 필자의 경우에는 Github Issue에서 제목부터 어떻게 지정하였는지에 따라서 문제 식별과 어느 저지 사이트에서 풀었는지 정의하도록 만들었다.

그리고 지정된 플레이어가 Github Issue에 댓글을 달았다면 자동적으로 제목에 기반하여 레포지토리에 자동적으로 코드를 커밋과 푸시를 하도록 설계하였다. 이 과정에서 지정된 플레이어 중에서 가장 먼저 댓글을 단 사람에게 1점의 스코어를 부여하도록 하였다.

## 점수 계산 방식

점수 계산 하는 알고리즘을 자동적으로 수행하기 위해서 `Github Action`을 통해 자동화 하도록 하였다. 그리고 게임 진행 방식에서 `Github Issue`에 `Comments`를 다는 것으로 점수와 자동적으로 코드를 업로드하는 것으로 정의하였으므로 `github-action.yaml`에는 아래와 같이 정의가 된다.

```yaml
on:
  workflow_dispatch: # for purpose of debugging and testing 
  issue_comment:
    types: [created]
```

누군가가 댓글을 담으로써, 자동적으로 이벤트가 발생하게 만들었으므로 이제 점수 측정해야하는 Issue나 최근에 등록된 Comments를 읽어서 `Repository`에 저장하고, `Score Board`에 점수를 갱신하면 된다.

필자의 경우에는 점수를 부여하기 위해 이미 점수가된 문제를 저장하고, 누가 이겼는지 기록하는 방식으로 구현하였다. 이유로는 누군가가 거짓으로 데이터를 조작할 수 도 있고, 추후에 누가 어떤 문제를 이겼는지 쉽게 알기 위해서 하나의 데이터베이스와 같은 형식으로 만들게 되었다.

`Score Board`에 점수를 갱신하기 위해 점수 갱신해야하는 점수를 하나의 텍스트 파일에 저장하여 조작이되지 않았음을 검증하기 위한 용도로(Commit 로그 분석용으로) 만들었다.

[승리자 목록](https://github.com/oMFDOo/whimpering/blob/main/scripts/win_list.txt), [스코어 보드 점수 판](https://github.com/oMFDOo/whimpering/blob/main/scripts/current_score.txt)

## 점수는 어떻게 출력하고 있나요?

솔직한 마음으로는 점수판은 지금 현재 `README.md`로 관리가 되고 있다보니, `README.md`파일을 통쨰로 변경해야한다. 그러나 필자의 경우에는 뭔가 Python 코드에서 `README.md`의 템플릿이 관리가 되는것은 무언가 매우 비효율적이라 판단하게 되었고, 그 결과 템플릿 엔진을 이용하고로 마음을 먹었고, `JinJa2`를 이용하게 되었다. 굳이 `JinJa2`를 쓸 필요가 없지만 추후에 `IF`나 `For`와 같은 복잡한 (만약에) 연산이 필요할 수 도 있으니 사용하게 되었다.

[템플릿 예제](https://github.com/oMFDOo/whimpering/blob/main/resources/template_score.md)

중요하지 않지만, 약간 필자는 `Go`언어로 `kubernetes` 제어하면서 `JinJa2`에 엄청 마음에 들었기 때문에 애용하고 있다. 그리고 최근에 진행한 프로젝트인 [MNIST from Keras H5 모델을 C++로 배포/컴파일](https://github.com/Piorosen/implement-mnist) 프로젝트에서도 C++ 코드 생성할 때 `JinJa2 Template Engine`을 이용하였다.

README.md를 생성해낸 뒤에 `Github Action`을 통해 자동적으로 커밋하여 이전의 README.md만 교체하면 점수는 변경할 수 있다.

## 어떻게 점수판을 그리고 있나요?

이 세상에는 `ASCII ART`에 진심인 사람이 정말로 많다. 그 예시로 `ASCII ART`를 만든 사람들 끼리 모여서 공유하는 사이트가 있을 정도로 [[ASCII ART Anime]](https://asciinema.org/explore) 인기가 많다. 아니면 `Terminal User Interface`라 해서 터미널 상에서 GUI 를 구현하는 사람이 있을 정도이다.

![](https://github.com/ArthurSonzogni/FTXUI/assets/4759106/6925b6da-0a7e-49d9-883c-c890e1f36007)

그렇기 때문에, 우리가 원하는 글자를 아스키 아트로 그리는 것은 매우 쉽고, 어떤 곳은 폰트 단위로 제공하는 곳이 있다. 그것은 바로 필자가 사용한 `pyfiglet` 라이브러리이며 공식 사이트는 [[사이트]](http://www.figlet.org/) 이다.

그래서 쉽게 `pip install pyfiglet`으로 설치하고 원하는 폰트를 찾고 `JinJa2`에 값을 넘겨주기만 하면 `README.md`는 완성이 되는 것이다. 그래서 실질적으로 코드는 라이브러리를 이용하여 호출 하였으므로 구조는 매우 단순하다.

```py
## 점수를 생성합니다.
# 기본적으로 left pad 를 적용하여, 최소 2글자로 표현되도록 만들었습니다.
def generate_score(score: int) -> str:
    art = pyfiglet.figlet_format(str(score).rjust(2, '0'), font="3d-ascii").rstrip("\r\n ")
    return art

## 이름을 생성합니다.
def generate_name(name: str) -> str:
    art = pyfiglet.figlet_format(name, font="doom").rstrip("\r\n ")
    return art

def get_current_information(file: str):
    with open(file, 'r') as reader:
        score = list(map(lambda x: int(x), reader.readline().split(",")))
        assert(len(score) == 2)
        winner = "Draw"
        
        if score[0] < score[1]:
            winner = 'Piorosen'
        elif score[0] > score[1]:
            winner = 'oMFDOo'

        return {"mfdo": score[0], "piorosen": score[1], "win": winner}

def generate_score_table(template: str, score) -> str:
    with open(template) as f:
        template = Template(f.read())

    result = template.render({
        'win_name': generate_name(score['win']),
        'mfdo': generate_name("oMFDOo"),
        'mfdo_score':  generate_score(score['mfdo']),
        'piorosen': generate_name("Piorosen"),
        'piorosen_score': generate_score(score['piorosen']),
    })

    return result
```

# 결과

안타깝게도... 만들고, 어떤 문제를 풀 것인지 정했지만, 여자친구가 최근에 면접이나 자소서를 쓰게 되면서 정작 이걸 쓸 일이 지금 당장은 없다는게 옥에티이다. 물론, 문제를 이슈에 만들고 댓글을 달면 자동적으로 `Repository`에 저장이 되는 것 까지 테스트 했다...

누군가가 추후에 이 글을 읽겠지만, 그 때에는 이 프로젝트가 쓰여졌었길 바란다...

+ 원래 `BackjoonHub`란 프로젝트와 연계해서 사용할 계획이였다만, 개인을 위한 프로젝트이다 보니 본 프로젝트에 적합하지 않았다. 다만, [[BackjoonHub 이슈]](https://github.com/BaekjoonHub/BaekjoonHub/issues/106#issuecomment-2154979678)를 발행하여 혹시나... 개인용이 아닌, 여러명이서 사용할 수 있으면 어떨지에 대해서 추가한 상황이다.

만약 누군가가 이긴다거나, 점수가 갱신된다면 아래와 같은 결과로 나타나게 된다.

![](/assets/img/post/2024-06-13-03.png)

# 후설

개발 시간은 무려 5~6 시간 정도 소요하였다. 누군가는 일회성 프로젝트인데 왜 이렇게 까지 구현하는지에 대해서 의문을 가질수 있겠지만, 필자의 경우에는 `재밌잖아?` 그리고 얼마전에 토익 시험에서 815점이 나와서 이제 백수인데, 6시간 정도는 태울만 하잖아? 라는 느낌이다. 또한, 블로그 포스팅이나 여자 친구와 함께 코딩 대결 할 수 있는 플랫폼 만드는건 `더 재밌잖아?` 원래 코딩은 재밌으려고 하는게 아니야? 라는 느낌으로 개발을 뚝딱 해버렸다.

요새 정말 다양한 프로젝트를 동시에 하고 있다. 가상 머신 부터 컴파일러까지 만들어보는 `https://github.com/DreamscapeVM` 프로젝트, 직접 `Keras to CPP for MNIST` 프로젝트인 `https://github.com/Piorosen/implement-mnist` 이 있다.

