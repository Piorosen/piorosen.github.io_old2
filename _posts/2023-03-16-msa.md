---
layout: post
title: MSA 구조를 이용한 백엔드 서버 개발
author: piorosen
categories: [Blogging, Cloud]
tags: [MSA, backend, go, microservice, kubernetes, nginx, ingress, load-balancer]
hide_title: false
---

# 개요

2022년 1월 부터 2022년 12월 까지 개발을 진행하였 던 족민의 달배란 프로젝트에서 겪었던 문제점과 개발 방향에 대해서 서술한다. 

족민의 달배는 음식점의 역경매 플랫폼이였으며, 일반 사용자가 원하는 음식을 담고, 여러 업체에게 요청하여 최저가에 입찰하는 구조이다. 즉 다나와 시스템에 있는 컴퓨터 역경매 시스템과 동일하며, 이를 음식점으로 들고온 사례이다. 거두절미하고 역경매 시스템을 개발하면서 겪었던 문제점과 미래에 까먹을 것을 예상하여 미리 작성하도록 한다. 본 글은 족민의 달배 프로젝트가 마무리가 되고, 3개월 뒤에 작성을 하였다.

# 서론

흔히 인터넷에서 MSA(Micro-Service Architecture)라 하여 기존 모놀로식(Monilithic Architecture) 아키텍처와 달리 큰 시스템에서 모든 기능이 동작하는 것이 아닌, 작은 시스템을 여러개 구성하여 동작하게 하는 것을 의미한다. 장점으로는 느슨하다, 독릭적이므로 여러 팀이 동시에 개발을 진행 할 수 있다. 코드가 망가지지 않는다 등등 장점이 많다고들 이야기 한다. 하지만 컴퓨터의 IT 세상은 항상 트레이드 오프 관계이다. 즉 무언가 이득이 되는 요소가 있다면 반드시 잃게 되는 요소가 있다는 것을 의미한다. 

모놀리식의 가장 큰 잠점으로는 데이터 송수신을 수행하기 위해 Memory 적으로 접근이 가능하다. 즉 A 데이터를 다른 서비스에게 전달하기 위해 메모리 참조 주소 값만 전달한다면 수행이 가능하다. 단점으로는 Scale Out이 정말로도 어렵다. 특정 시스템만 100개의 컴퓨터로 구성하고, 부하가 작은 시스템에는 1개의 컴퓨팅 자원을 할당하는 것이 어렵다는 것을 의미한다.

MSA의 구조에서 가장 큰 장점으로는 Scale Out이 가능하며, 여러 개발자가 동시에 작업하더라도 문제가 발생하지 않는다(물론 REST API를 통해 구성한다면 문제가 발생하지만, gRPC와 CI/CD가 구성된 환경이라면 크게 문제가 발생하지 않는다. 이유는 gRPC의 특성으로 이해하면 된다.). 단점이라면 흔히 테스트 코드 및 Mock 서버 구성(테스트 코드나 Mock 서버는 외부와 통신하는 것에 중점이 되어 있지 않기 때문에)에 복잡함이 있다고 이야기하기도 한다. 또한 다른것은 시스템의 복잡성이 올라가게 된다는 것이 매우 크다.

# 본론

족민의 달배란 시스템에서 마이크로서비스 아키텍처를 도입하고, 시스템 배포를 위해 Github Action을 통한 CI와 ArgoCD를 통한 CD를 구성하였다. 그리고 프론트엔드는 React에서 Nginx로 빌드하여 배포하여 미리 빌드된 Server Side Rendering을 구현하였다. 그 외에 백엔드는 Nginx-Ingress를 통해 로드밸런서를 궇녀하고, 각 내부 Deployment와 연결하여 시스템을 구성하였다.

그리고 HTTPS와 같은 TLS 인증서를 위해 Cloudflare를 통해 구현하였다. 외부 시스템의 구성은 아래와 같다.

1. Cloudflare를 통한 1차적인 CDN 및 1차 로드 밸런서
2. Synology NAS를 통한 역방향 프록시 (특정 URL로 들어오는 입력을 다른 IP, 포트로 전송함.)
3. 프론트엔드의 경우
   1. Kubernetes의 Service 요소 중 하나인 NodePort를 통해 연결됨.
4. 백엔드의 경우
   1. Kubernetes의 Nginx-Ingress를 통하여 로드밸런싱
   2. Nginx-Ingress는 Nginx가 기반이므로 L4(흔히 스트림이라 이야기함), L7 로드밸런싱이 가능함(그렇지만 L7 로드밸런싱을 선택함)
   3. URL 기준 로드밸런싱을 수행함

프록시와 역방향 프록시는 본 글에서 중요한 요소가 아니므로 생략한다. 그렇지만 간단하게 설명을 하자면 역방향 프록시은 서버단에 적용되는 프록시이며, 일반적인 프록시는 사용자의 PC에 사용된다.

이때 시스템의 구성을 Microservice 마다 MongoDB를 두거나, 하나의 역할 단위로 시스템을 구성하였었다. 예시로, 로그인 서버, 관리자 서버, 쿠폰 및 포인트 관리 서버, 역경매 등록 서버 등 다양하게 분리하여 시스템을 구성하였다.

확실하게 느낀 장점으로는 시스템이 분할이 되면서 한 곳의 코드만 확인만, 유지 보수가 확실하게 될 것이라 생각하게 된다. 즉 ```객체 지향 시스템을 구성할``` 때 최고라고 생각하게 되었다. MSA를 다시 다르게 생각한다면 객체 지향 프로그래밍을 시스템론적으로 들고온 개념이지 않을까? 란 생각을 하게 되었다. 기본이 모든 시스템이 하나로 작성 되던 것을 OOP란 개념을 도입하듯이 시스템 또한 객체 지향적인 시스템을 구성하고자 탄샌항 개념이 MSA가 아닌지 되돌아보게 되었다.

# 정리

결론적으로 이야기하자면 MSA 구성하였지만 유지보수적인 개념으로 보자면 실패하였다. 그렇지만 MSA가 반드시 최고가 아닌 하나의 방법론이라는 것을 알게되는 과정과 OOP와 같이 객체지향 시스템이란 생각을 하게 되는 기회가 되었다. 물론 추후 미래에 MSA 관련 디자인 패턴이 등장하면서 시스템의 강제화 및 솔루션이 등장할 것이라 생각한다.

이러한 생각을 하게 된 이유는 과거에 디자인 패턴으로 작성한 어떤 소프트웨어를 개발하고, 2-3년 뒤에 새로운 기능 추가 업무를 하달 받은 적이 있다. 업무는 "파일 시스템에 로그를 저장하고, 이전 로그를 시각화하는 기능"에서 파일 시스템에서 데이터베이스로 동작하도록 마이그레이션 하는 업무였다. 그렇다면 모든 시스템이 변경이 되어야 하지만, 추상화와 디자인 패턴으로 인한 코드 레벨로 강제화가 되어 있어 코드의 가이드라인에 맞춰 작성만 하면 되었기에 업무가 엄청 간단하게 변경이 되었었다.

지금 현재는 MSA는 분명히 좋지만, 모두가 이해가 사전 이해도가 있어야하며, 대규모 프로젝트가 아니라면 한번더 고민하게 되는 사례가 될 것이라 생각한다. 분명히 대규모 시스템에서 MSA로 구성한다면 Scale Up이 아닌 기능 별 Scale Out에 용이 할 것이기 때문이다.

# Yaml 관련

Nginx의 Ingress는 Baremetal(kubeadm)을 기준으로 작성이 되었다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: moonplanet-ingress
  namespace: ingress-nginx
  annotations:
    nginx.ingress.kubernetes.io/configuration-snippet: |
           more_set_headers "Access-Control-Allow-Origin: $http_origin";
    nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
    nginx.ingress.kubernetes.io/cors-allow-methods: PUT, GET, POST, OPTIONS, DELETE, PATCH
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  ingressClassName: ingress-nginx 
  rules:
  - http:
      paths:
      - path: /oauth(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: moonplanet-location-service
            port:
              number: 8080
      - path: /location(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: moonplanet-location-service
            port:
              number: 8080
      - path: /restaurant(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: moonplanet-location-service
            port:
              number: 8080
      - path: /reward(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: moonplanet-location-service
            port:
              number: 8080

...

```
