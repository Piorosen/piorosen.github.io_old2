---
layout: post
title: Cloudflare의 API 활용하여 서브도메인의 SSL 인증서와 DNS 관리
author: piorosen
categories: [Blogging, Cloud]
tags: [cloudflare, ssl, subdomain, api, macro]
hide_title: false
---

# 개요
최근 동아리방 서버를 구축 하기 위해 OSI 7계층과 쿠버네티스, DNS 서버에 대해서 공부를 했었다. 그러면서 문득 과거 2021년 3월에 진행 했던 [웹페이지](http://directfyou.com) 외주를 새롭게 맡는다면 어떻게 시스템을 업그레이드를 할 수 있을까? 란 고찰을 하게 되었다.

2021년 3월에 진행했던 외주는 서브 도메인에 여러개의 하위 업체의 페이지를 만들고자 하였고, 관리자 페이지에서 하위 업체의 페이지를 관리가 가능한 시스템 구축을 원하였으며, 아래의 이미지 처럼 하위 업체 페이지와, 관리자 페이지를 통하여 만들었었다.

하위 업체 페이지 예시|관리자 페이지 예시
:---:|:---:
![이미지1](/assets/img/post/2021-10-26-subdomain.png)|![이미지1](/assets/img/post/2021-10-26-manage.png)

하지만 해당 시스템에서 가장 큰 결함, 문제가 하나 있다. 

> 1. Cafe24에서 서브 도메인을 추가 할당및 서버와 연결 한다.
> 2. 외주 관리자 페이지에서 서브 도메인에 대한 응답을 처리 하도록 관리 페이지에 추가 한다
> 3. Cafe24에서 서브 도메인은 최대 50개 연결만 가능하다.

위의 내용 처럼 매번 서브 도메인을 할당 할 때, 삭제 할 때 마다 번거로운 2중 작업이 필요 했으며, Cafe24의 특성상 SSL을 추가 하려면 1개의 서브 도메인 마다 1년에 20만원 정도의 비용이 발생 한다. 그래서 만약 위의 작업을 1번의 작업으로 가능한지, 서브 도메인의 수를 무제한의 한도로 만들기 위해서 어떻게 해야할지 고민, 고찰을 해보았다.

# DNS 서버

Cafe24의 경우는 따로 API를 제공하지 않기 때문에 수동으로 DNS 서버의 값을 변경을 해야한다. 그렇다면 DNS 서버를 API 형식으로 제공을 해주는곳으로 이관 한다면 통합이 가능하지 않을까? 란 생각을 한다면 이야기가 달라진다. 이것은 정말 놀라운 생각이다. 제한적인 Cafe24를 사용하는 것이 아닌 다른 DNS 서버를 찾아보니 가장 흔하며 대표적인 [Cloudflare](https://www.cloudflare.com/)이다. Cloudflare를 활용하게 된다면 DNS 서버와 함께 프록시 서버를 통하여 SSL 인증서 까지 제공하기 때문에 SSL 비용도 절감이 가능하다!

<img src="https://www.cloudflare.com/img/logo-cloudflare-dark.svg" alt="cloudflare logo" width="60%"/>
{: .text-center }

Cloudflare는 DNS 서버이기도 하며, CDN 이나 라우팅, 로드밸런싱과 같이 여러 서비스를 제공하는 업체이다. 도메인 구입은 Cafe24에서 하더라도 DNS 서버는 Cloudflare를 사용해서 서브 도메인에 관한 처리를 하는 것 이다. Cloudflare는 API 형식으로 DNS를 조작이 가능하며, 프록시를 통하여 SSL 인증서 까지 제공을 해주니 위의 결함을 모두 해결이 가능하다.

# 만약 다른 웹페이지 개발을 한다면?

새롭게 개발을 해야 한다면 Cloudflare의 API를 호출을 통해 새로운 CNAME 레코드를 만들어 새로운 서브 도메인을 만든 뒤, 관리자 페이지에서 일괄 적용하도록 변경을 할 것이다.

[Cloudflare API 명세서](https://api.cloudflare.com/#getting-started-requests)를 읽은 뒤 만들지 않을까 싶다!

# 마무리

과거에는 기술이 없어서 구현이 불가능 하거나, AWS의 EC2 서비스를 구매하여 NginX, Apache를 직접 건들여서 L7 로드 밸런싱이 필요하다고 생각을 했었다. 하지만 Cloudflare를 통하여 DNS 로드 밸런싱이란 것을 배웠으며, 프록시 서버를 통하여 비 암호화 HTTP 통신을 HTTPS 로도 변환이 가능한것을 알게되었다.

그 덕분에 내부망 통신은 전부 http를 진행하고, 외부와의 소통을 할 때 따로 리버스 프록시 서버를 구축하여 SSL 인증서를 관리를 하는 방법도 최선의 방법이 되지 않을까 싶은 생각을 한다.