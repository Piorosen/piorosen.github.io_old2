---
layout: post
title: Minimax 알고리즘 공부 및 내용 점검
author: piorosen
tags: [algorithm, ai, minimax, gomoku]
hide_title: false
categories: [Blogging, Develop]
# feature-img: "assets/img/feature-img/"
# feature-img: "http://placehold.it/700x300"
# thumbnail: "assets/img/thumbnails/feature-img/"
# thumbnail: "http://placehold.it/200x200"
---

# 개요
교수님께서 DQN(Deep Q-Network) 와 같이 Q 함수를 이용하여 강화 학습을 이용하여 오목 프로그램을 만들어 보자고 하셨다. 그렇지만 Tensorflow나 PyTorch와 같은 딥러닝 라이브러리를 사용하는것이 아닌 직접 코드를 작성 해서 만들어 보자고 하셨다. 그래서 게임에 적용할 게임 인공지능에 대해서 공부를 하게 되었다. 최종 목표로는 강화 학습을 직접 구현 해보고, 오목 인공지능을 만들어 보는 것이다. 인공지능에 대해서 공부를 하면서 minimax 알고리즘 이라는 최소극대화 알고리즘 을 공부하게 되었다.

# Minimax 알고리즘 이란?

Minimax 알고리즘은 현재 상태를 입력 받았을 때, 점수를 나타낼 함수가 반드시 꼭 필요하다. 그리고 Tree란 개념이 사용이 되므로 Depth, 트리의 깊이 변수도 필요하다.

```python
# Work
1. 현재 게임 보드 상태의 점수를 구함.
2. 다음에 둘 수 있는 모든 수를 분기(Branch) 한 뒤 각 분기의 점수를 구함.
3. '2번'에서 분기 했던 수를 또 다시 한번 더 분기 한 뒤 점수를 구함.
4. N번 반복 함.
5. Minimax 알고리즘으로 최소극대화 점수를 구함.
```

Minimax 알고리즘은 2명이서 즐길 수 있는 게임에서 주로 사용이 된다. 그리고 Minimax 알고리즘에서 최소극대화를 조금 알기 쉽게 표현을 하자면
```python
1. 나의 턴일 때 최고의 수를 두어야 하는 상황일 때
2. 상대방 턴일 때 최고의 수를 두어야 하는 상황일 때
총 2가지의 상황이 있다.

나의 턴일 때는 최고의 점수를 가지는 수를 두어야 하고, 상대방 턴일 때도 마찬가지로 이기기 위해서 최고의 점수를 가지는 수를 두게 될 것이다.
이 때 나의 턴이 최고의 점수를 가지면서 상대방도 최고의 점수를 가지게 될 때 나의 턴의 점수가 최대한 높은 수를 두는 알고리즘 이다.
```

# 코드 설명

여기서 코드를 설명 할 때 모두 전부 구현이 된 것이 아닌, 이해를 하기 위해서 일부만 구현을 하였습니다. 예를 들어 board::getScore 함수는 구현하지 않았습니다.

```cpp
// 현재 상태를 표현이 가능한 보드판.
struct board {
    char map[13][13];
    
    // 현재 보드 상태에 따른 스코어 점수.
    int getScore() {
        return 0;
    }
};

// 트리 구성을 위해서 node 를 만듬.
struct node {
    board current;
    vector<node> next = vector<node>();
};
```

여기서 현재 보드 상태와 보드 상태를 분기가 가능하게끔 (Tree로 나타내기 위한) Node 구조체를 만들었다.

```cpp
// 현재 보드 상태에서 착수가 가능한 상태를 계산 함.
vector<node> canNextStep(board b) {
    vector<node> result = vector<node>();
    
    // 단순 10번 반복한 데이터를 넣음. 여기서 보드 데이터와 착수 위치 데이터 까지 같이 넣을 수 있음.
    // 그렇게 된다면 board를 반환 하는것이 아닌 point로 또한 가능 함.
    // 여기서 오목이면 렌주룰과 같은 룰 체크 기능을 넣을 수 있음.
    for (int i = 0 ; i < 10; i++){
        node n = node();
        n.current = board();
        result.push_back(n);
    }
    return result;
}

// 분기 가능한 보드를 모두 분기 함.
node makeRootNode(board b, int depth, int curDepth = 0) {
    node parent = node();
    
    // depth 변수 만큼 착수 지점을 분기 함.
    parent.current = b;
    parent.next = canNextStep(b);
    
    if (depth == curDepth) {
        return parent;
    }
    
    for (int i = 0; i < parent.next.size(); i++) {
        parent.next[i] = makeRootNode(parent.next[i].current, curDepth + 1);
    }
    
    return parent;
}
```

여기서 트리 구조로 N-Depth 만큼 착수를 할 가능성이 있는 모든 수를 분기 하여 트리 구조로 만든다.
트리 구조로 만들게 되었다고 하면 여기서 minimax 알고리즘을 이용하여 나와 상대방이 모두 최고의 수를 두었을 때 내가 가질 수 있는 최고의 수를 내야 한다.

```cpp
node minimax(node root, bool isMax = false) {
    if (root.next.size() == 0) {
        return root;
    }
    
    // isMax란 변수로 최대 최소를 구함.
    // isMax가 false이면 현재 보드 분기 목록 중에서 점수가 가장 낮은 게임 판을 구함.
    // isMax가 true이면 그 와 반대로 점수가 가장 높은 게임 판을 구함.
    node value;
    for (int i = 0; i < root.next.size(); i++) {
        node tmp = minimax(root.next[i], !isMax);
        
        if (isMax && value.current.getScore() < tmp.current.getScore()) {
            value = tmp;
        }else if (!isMax && value.current.getScore() > tmp.current.getScore()){
            value = tmp;
        }
    }
    
    return value;
}
```

코드 상으로는 매우 간단하다. 여기서 코드는 재귀로 돌아가면서 isMax는 매번 not 연산자로 바뀌고 있다. not을 한 이유는 이 게임은 턴제로서 상대방이 한 수를 둔 뒤 다른 사람이 또 한 수를 두기 때문이다. 그러므로 isMax가 true이라면 인공지능이 둘 차례 라는 의미이며, false이면 인공지능이 아닌, 상대방이 두는 차례 라는 의미이다. 각각 isMax에서 최소값과 최대값을 구한다. 구하면서 최종 Root 노드에서 인공지능이 이길 수 있는 최대값을 내도록 한다.

```cpp
// 다음 수를 계산 함.
board nextStep(board b) {
    // 현재 상태를 기준으로 보드를 분기 함.
    node tree = makeRootNode(b, 5);
    // 분기한 보드 중 최고의 수 노드를 구함.
    node bestNode = minimax(tree);
    
    // 그리고 최고의 노드를 반환 함.
    return bestNode.current;
}
```

현재 보드 상태를 기준으로 노드를 분기 하고, 분기한 노드 중 인공지능이 둘 수 있는 최고의 노드를 구한 뒤 반환을 하는 함수이다.
현재는 board 상태만 반환을 하지만 board 구조체 안에 point 라는 자료를 추가하여 착수 한 위치도 기억하게 할 수 도 있다.

# 알파 베타 프루닝

여기서 현재 minimax는 모든 가능 한 수를 분기 한 뒤 최 하위에 있는 노드의 점수를 구하여 최 상위 노드에서 결과를 도출 하는 방식이다. 

하지만 해당 방식에서는 19x19 크기의 바둑판 경우에서는 매 수 마다 361개의 노드가 분기가 되므로 10번 째 후의 수를 37,589,973,457,545,958,193,355,601 개의 노드를 분기하게 되므로 매우 비효율 적이다. 그래서 A* 처럼 최고의 수가 될 가능성이 낮은 노드는 분기하지 않도록 해서 최적 해를 구하는 알고리즘이다.

# 개인 생각

minimax 알고리즘에 대해서 공부를 하게 되면서 다양한 알고리즘을 알게 되는 경험이 되었다. 예를 들어서 "알파-베타 프루닝" 이나 "몬테 카를로 트리 서치(MCTS)" 처럼 다양한 게임 이론에 쓰이는 인공지능을 알게 되었다. 공부를 하게 되면서 결국 나의 목표는 DQN 처럼 매번 게임을 하면서 학습을 하는 인공지능을 만든다고 했었는데 그 해결책이 MCTS 가 될 것 같은 느낌이다. MCTS는 정말 간단한 이론 이지만 안에 들어 있는 내용은 포인터와 같이 응용이 가능한 범위가 엄청 넓은 알고리즘인 것 같다.