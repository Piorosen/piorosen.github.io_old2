---
layout: post
title: 동적 메타 헤더를 위해 리액트 코드에서 PHP 코드로의 전환
author: piorosen
categories: [Blogging, Develop]
tags: [ssr, csr, rendering, php, og, react, build, vanilla, react-helmet]
hide_title: false
---

# 사전지식

우선, 상황에 대해 설명이 필요한데, 현재 개발을 진행하면서 기존에 구매 및 사용하던 서버가 PHP가 있었기에 다른 서버로 이전 및 전환이 불가능한 상황이였음을 이해가 필요하다. 따라서, 왜 Node.JS 서버가 아닌 PHP 서버를 쓰고 있는가에 대한 의문에 대한 대답은 서버를 추가적으로 구매할 수 없는 상황과 기존에 있는 PHP 서버를 활용해야했기 때문이다. 이러한 이해관계를 이해하고 있다면 React로 개발한 내용을 PHP 서버에 올렸는가에 대한 의문은 해결이 될 것이다.

# 개요

본론으로 돌아와서 새로운 신규 개발자가 PHP로 작성된 프론트엔드 코드를 React로 이전 및 전환을 수행하였다. 하지만, 무엇이든지 무탈하게 지나가면 좋지만, 리액트는 Client Side Rendering(CSR) 방식이고, PHP는 Server Side Rendering(SSR)이면서 생긴 문제점이 발생하였다. 현재 동일한 코드에서 서브 도메인마다 내부적인 라우팅을 통해 페이지를 생성해내는 방식을 사용하고 있었다. 하지만 CSR 방식으로 전환하면서 OG 메타 헤더에 문제가 발생하게 되었다.

OG 메타헤더는 사용자가 클릭하기 전에 크롤러가 데이터를 수집하여 미리 사용자에게 보여주는 것이다. 또한, 크롤러의 보안 정책상 Javascript는 실행하지 않고, 오직 단일 페이지 정보만 수집하여 데이터를 생성한다. 그렇기에, CSR에서 동적으로 메타 헤더를 변경하더라도, 웹 브라우저에서는 변경이 되지만 크롤러에서는 변경이 되지 않는것이다.

## 상황 정리

요약하자면 현재 상태에서는 서브 도메인 마다 OG 메타 헤더가 변경이 되어야 하는 상황에 CSR 방식인 리액트는 적합하지 않았다. 이유는 리액트는 "클라이언트" 중심의 렌더링 방식이기 때문이다. 물론, 리액트를 PHP 서버에 올리기 위해 리액트를 사전 빌드하여, `CSS`, `JS`, `HTML`로 변환하여 배포하였다.

[https://stackoverflow.com/questions/77020735/how-do-i-generate-dynamic-meta-description-and-image-in-react-using-react-helmet](https://stackoverflow.com/questions/77020735/how-do-i-generate-dynamic-meta-description-and-image-in-react-using-react-helmet)

# 아이디어

어떻게 해결하면 좋을까란 회의를 진행하면서 기발한 아이디어가 나의 머릿속에 스쳐지나갔다. 해결 방법이 떠올랐지만 주변에 `혹시,, 아이디어 있어? 나 해결할 방법이 났지만 코드가 많이 이상해질거야 괜찮겠어?` 라고 질의를 한 결과 `좋아요!`란 답변을 받았다.

먼저 이야기 하자면, `React.JS` -> `Building HTML with JS code` -> `Inject PHP code into React code builded` 과정을 거치면서 코드를 많이 더럽혔다.

아래의 코드는 `React.JS`에 있는 `index.html` 코드의 일부를 발췌한 내용이다. `React` 를 빌드하면서 코드를 압축하므로, 해당 코드가 가능한 것이다.

![](/assets/img/post/2023-11-12-01.png) 

# 마무리

신난다! 코드 가독성은 떨어지고, 리액트와 PHP 코드가 함께 존재하는 신기한 코드이지만, 일단 정상적으로 동작한다는 것을 알 수 있다.

또한, 하나의 문제를 해결하기 위해서 불가능한 것을 가능케 하는 것이 아닌, 한발 뒤에서 지켜보면서 어떻게 해결할 것인지 큰 그림을 보는것이 중요하다.

솔직하게, PHP 서버에서 `React.JS`를 사용하기 위해서 `React.PHP` [[React.PHP]](https://reactphp.org/) 도 고민했을 정도였다.

즉, `React.JS` 에서 동작 안하넹... 어떡하지란 고민에서 더 나은 차선책이 나오면서 `React.JS`를 빌드하고 나온 결과물을 PHP 코드로 한번 더 변경하여 문제를 해결하는... 신박한 구조로 구현하게 되었다.





