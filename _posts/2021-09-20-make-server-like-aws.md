---
layout: post
title: DNS 서버의 SRV를 활용한 AWS-Like 서버 구축
author: piorosen
tags: [osi, layer4, layer7, aws, dns, dhcp, k8s, haproxy, nginx, https, srv, ssh]
hide_title: false
# feature-img: "assets/img/feature-img/"
# thumbnail: "assets/img/thumbnails/feature-img/"
categories: [Blogging, Develop]
---

# 개요

내년, 2022년에 학과 동아리 회장과, 동아리 서버 컴퓨터 관리를 맡게 되었다. 그리고 동아리 서버 컴퓨터를 현재 활용을 하고 있지 않고, 연구 과제를 진행 할 때, 딥러닝이나, 서버를 돌릴 때 가끔 사용 하고 있었다. 그렇지만 교수님은 컴퓨터 리소스는 많고, 잘 사용하지 않으니 일반 학생도 쉽게 사용할 방법을 찾아보라고 하셨다. 

<img src="/assets/img/post/2021-09-21-server.JPG" alt="Server" width="60%"/><br>[서버 컴퓨터 모습]
{: .text-center }



그래서 컴퓨터가 여러대 이면서, 여러명이 동시에 사용할 방법을 생각을 하다가 Amazon Web Service 처럼 구축을 한다면 현재 교수님이 운영 중인 [학과 Judge 사이트](http://ascode.org)나, 교육용 서버를 전부 옮겨서 일괄 관리를 한다면 좋지 않을까? 라는 생각을 하게 되었다. 그래서 AWS 처럼 클라우드 컴퓨팅 서비스를 직접 구현 해보고자 하였다.

# AWS에서 서버 동작 원리

클라우드 컴퓨팅 서비스를 구현 할 때 구현 모델이 이미 있으니 공부 할 방향이 제대로 잡혔었다. GCP나, Azure 이 3가지 중에서 왜 하필 AWS를 학습 및 구현 모델로 잡은 이유는 현재 AWS는 국내, 해외에서 점유율이 상당히 높기 때문이였다. 

<a href="https://aws.amazon.com/"><img src="/assets/img/post/2021-09-21-aws.png" alt="AWS" width="60%"/></a><br>
{: .text-center }

Amazon Web Service에서 EC2(Elastic Compute Cloud)에서 SSH 서버를 접속 하는 방법은 AWS의 Public DNS서버 주소를 입력하면 된다고 되어 있다.

> AWS 서버에서 SSH 접속 하는 방법 <br> 
> [Connect to your Linux instance using an SSH client](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

여기서 처음 해당 문서를 읽었을 때 DNS 서버를 활용 해서 사용자에게 보여주어야 하는구나! 라는 생각을 하게 되었다. 지금 현재는 이해를 하고 글을 쓰지만 이해하기 전에는 Reverse Proxy나 NginX을 이용해서 L4 스위치를 통해서 해야하는가..? 라는 생각을 하기도 했다.

<img src="/assets/img/post/2021-09-21-dns-server.png" alt="DNS Server" width="60%"/>
{: .text-center }

현재 이해한 방식으로는 Public DNS가 사실 DHCP Server 와 같이 오랫 동안 사용하지 않으면 IP를 회수 하고, 사용 요청을 하면 새롭게 IP를 할당 하는 방식이 아닌지 생각을 하게 되었다. 그렇게 된다면 Public DNS 서버에 접속 할 때 이미 할당이 되어 있는 IP가 있다면 해당 IP로 접속을 하게 되고, 만약 오랫동안 DNS 서버에 접근을 안하거나, 해당 IP가 사용이 되지 않는다면 회수를 해, 새로운 서비스에 제공을 하는 역할을 하고 있는게 아닐까 생각하고 있다.

그래서 검색을 해본 결과 [Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)이라는 결과가 있었으며, [EC2](https://aws.amazon.com/ko/ec2/pricing/on-demand/#Elastic_IP_Addresses)에서는 1개가 기본 제공이다. 그래서 Elastic IP가 Public DNS에서 IP를 관리를 하고 EC2에서 접근 하는 방식이라고 생각하게 되었다.

# 어떤 삽질을 했는가?

처음에는 끔직할 정도로 이상한 생각을 많이 했다. 아무런 공부를 하지 않고 서버를 구축 하려고 했지만 엄청나게 많은 막막함과 어려움이 존재 했었다. 그래서 어떠한 공부를 해야 할까? 생각을 하다가 네트워크 기본 부터 해야겠다는 생각이 들었으며, 학교 공부 할겸 학교 책으로 네트워크 공부하게 되었다. 책 내용은 네트워크 이지만 자세한 내용은 OSI 계층에 대한 공부 였다. L1 부터 L7 까지의 내부 동작 프로토콜에 대한 내용 설명 이였지만 최소한의 기초지식이 있어서 이해하기가 편했다.

공부할 때 AWS에서 Public DNS 를 이용해서 무언가를 처리를 하고, AWS 서버에서 내부적으로 로드 밸런싱을 통해서 EC2 에 접근을 했을 것이다! 란 생각을 했었다. 그래서 L4 로드 밸런싱과 L7 로드 밸런싱에 대해서 생각을 하게 되었고, NginX에 대해서 공부를 하다가 Domain 기반으로 Reverse Proxy를 하려고 하였으나, Reverse Proxy와 HTTP를 공부를 더 해보니 L7 로드 밸런싱이라는것을 알게 되었다. 처음에는 어떻게.. 도메인으로 SSH 를 접속이 가능할까..? 란 생각을 했었지만 결국엔 이 도메인이란 정보는 HTTP 에 정의가 되어 있고, 도메인은 DNS 서버에서 IP로 변경이 되고, HTTP 요청 할 때 도메인과, IP를 같이 보내는 것 이였다. 

그래서 어떻게 해야하는가? 라는 생각을 하게 되었고, L4 에서는 TCP, UDP, IP, Port 정보만 있다고 생각을 했을 때, 할 수 있는 방법은 포트포워딩이 최선의 방법이라 생각하게 되었다. OSI 7 계층에 대해서 공부를 하다가 L3, L4, L7 의 역할을 더 자세히 알게 되었고, 각각의 로드밸런싱을 어떻게 해야하는지 알게 되는 순간이였다.

# 그래서 어떻게 해야하는가?

학교에서 제공 받은 IP 주소는 제한적이며, 유한적이다. AWS 처럼 Elastic IP 를 구현에는 문제가 상당히 있다. 그래서 구현을 한다고 하면 L4 스위치 기능을 활용 해서 포트포워딩 개념으로 구현 해야 한다. 그렇지만 최대한으로 DNS를 활용을 한다면 IP와 Port를 같이 반환하는 SRV 레코드가 있다. 이 SRV 레코드를 활용해서 L4 스위치 기능을 활용이 가능하다. 그래서 생각을 해낸것이 DNS 서버에서 SRV 레코드를 활용 해서 구현 하기로 생각 했다. 이때 자체적인 DNS 서버를 구축해서 아래의 도메인으로 요청이 들어오면 22번 포트로 데이터를 보낸다.

> username.ssh.testdomain.com <br>
> 로 들어오면 ssh에 관련해서 L4 로드 밸런싱을 진행하고, <br>
> username.http.testdomain.com <br>
> 로 들어오면 http 관련해서 L7 로드 밸런싱을 진행하도록 한다.

이렇게 구현을 한다면 DNS 서버에서 HTTP 로 요청하면 NginX 로 로드밸런싱을 진행하는 서버에 요청 하고, ssh로 요청하면 L4 스위치로 로드밸런싱을 진행하도록 한다.

# 와! 이것이 AWS

비록 AWS의 Elastic IP와 같은 기술을 활용 해서 IP관리 시스템을 도입하고 싶지만 제한적이고 많은 양의 IP를 미리 할당을 해 놓고 있지 않았기 때문에 Elastic IP를 구현하기 어렵다. 그래서 IP를 할당하는것이 아닌, L4 로드 밸런싱을 통해서 진행 하고자 했다. 그래서 DNS 서버의 SRV 레코드를 활용해 IP와 Port를 받아서 로드 밸런싱 하고자 했다. 이때 도메인에 특정한 규칙을 만들어 IP와 포트를 반환하게 하는 형식으로 만들어 사용하는 방법 또한 구현이 가능 할 것 같다.



