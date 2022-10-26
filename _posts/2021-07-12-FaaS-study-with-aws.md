---
layout: post
title: Amazon Web Service에 대해 공부하다 알게 된 Cloud Service별 가상화 단계
author: piorosen
categories: [Blogging, Develop]
tags: [aws, lambda, faas, serverless]
hide_title: false
# feature-img: "assets/img/feature-img/"
# feature-img: "http://placehold.it/700x300"
# thumbnail: "assets/img/thumbnails/feature-img/"
# thumbnail: "http://placehold.it/200x200"
---

# Cloud Service와 각 서비스별 가상화 레벨

과거 시스템을 직접 구입 해서 시스템 구축을 진행을 하였으며, 웹 페이지 경우에는 호스팅(Hosting) 업체를 통하여 쇼핑몰이나 수강 신청 사이트를 만들었다.

현재는 컴퓨터를 직접 구입을 해서 인프라 부터 시스템까지 구축을 하는것을 온프레미스(On-Premises)라 부르고 있고 그 외에 클라우드 서비스를 제공하는 기능에 따라서 이름이 다르게 불려지고 있다.
 - On-Premises
 - IaaS (Infrastructure As A Service)
 - PaaS (Platform As A Service) 
 - CaaS (Containers As A Service)
 - FaaS (Function As A Service)
 - Baas (Backend As A Service)
 - SaaS (Software As A Service)

[![이미지](/assets/img/post/2021-07-12-service-list.png)](https://blog.oio.de/2018/02/14/function-as-a-service/)
각 서비스 별 가상화 단계 (흰색 : 가상화 레벨, 파란색 : 구현 레벨)
{: .text-center }

### 1. IaaS (Infrastructure As A Service)
On-Premises와 가장 유사하며 쉽게 이전이 가능한 서비스가 IaaS 이다. 그 이유는 위에 있는 이미지 처럼 클라우드 업체에 있는 CPU, Storage, Network와 같은 서버 리소스를 할당을 해주는것이 바로 IaaS 이기 때문이다.

이해하기 쉽게 이야기를 하자면 클라우드쪽으로 이야기를 하자면 AWS(Amazon Web Service)의 EC2(Elastic Compute Cloud
) 이며, 프로그램으로 설명을 하자면 Virtual Machine 개념이다. Virtual Machine을 보게 된다면 하나의 컴퓨터를 리소스를 가상화 한 뒤 여러개의 OS(Operating System)을 설치를 하는것이다.

즉 IaaS의 경우에는 부트 로더나 바이오스 부분과 같은 하드웨어와 연관 되어있지 않는 소프트웨어 영역을 가상화 해서 제공 해주는 서비스 이다.
개발자는 운영 체제, 애플리케이션, 미들웨어 등을 관리하는 반면 제공업체는 모든 하드웨어, 네트워킹, 하드 드라이브, 스토리지 및 서버를 관리하며 가동 중단, 복구, 하드웨어 문제를 담당합니다.


### 2. PaaS (Platform As A Service)
인프라 구축이 아닌 플랫폼을 서비스를 제공 해준다. 즉 애플리케이션 개발 환경을 서비스 제공해주는 모델이다.

예를 들어서 .Net Framework나 Apache-Php-Mysql을 하나로 묶어서 제공 해주는 Xampp가 될 수 있을 것이며, 나의 개인적인 생각은 docker-compose도 해당 내용이 될 수 있지 않을까 라는 생각을 해본다.

docker-compose 가 PaaS라고 생각하는 이유는 "여러개"의 컨테이너를 연관 관계를 작성 하여 개발 환경을 구축 하거나, 서버를 구축해서 제공이 가능하기 때문이다.

[Apache, Php, Mysql, PhpMyAdmin, VS Code](https://github.com/Piorosen/apm-with-vscode-dockerized)를 통합하여 웹 개발 환경을 구축한 예.
즉 개발 환경을 구축 하여, 해당 개발 환경 안에서 시스템을 구축 한다.

IaaS 경우 여러개의 개발 환경이 구축이 가능 함.(.Net Framework와 Django 설치 등.)

### 3. CaaS (Containers As A Service)
CaaS 에 대해서는 공부를 진행 하고 있지만 CaaS 의 형태로 서비스 하고 있는 소프트웨어, 업체를 나열 한다.
 - AWS(Amazon Web Service)의 ECS(Elastic Container Service)
 - Google의 Cloud Run
 - k8s(Kubernetes)
 - docker

나의 생각으로는 도커 처럼 하나의 프로그램의 가상화를 서비스 해주는 것이라 생각 한다. 도커는 환경에 영향을 받지 않고 배포와 확장에 자유롭기 때문이다.

### 4. FaaS (Function As A Service)
현재 공부 중인 내용 중 하나이다. FaaS는 내용 그대로 Function As A Service, 즉 기능을 서비스를 제공 한다. FaaS의 대표적으로는 AWS의 Lambda이며, 다음에 작성할 내용 중 하나이다.

올해 2021년 K 해커톤을 진행 하면서 백엔드를 AWS의 Lambda를 이용할 예정이며, Database로 NoSQL인 DynamoDB를 이용할 예정이다.

간단하게 요약을 하자면 StateLess 이며 ServerLess라고 부를 수 있다.
Stateless는 상태를 저장하지 않는다는 의미 이며, 실행이 된 후 모든 데이터는 소멸이 된다. 그래서 상태를 기억하고, 데이터를 읽어들일 Database나 Storage 가 반드시 필요하다.

그리고 FaaS는 요청이나, 호출 하는 것이 아닌 이벤트 형식으로 트리거가 된다면 내부 기능, 함수가 호출이 되는 방식이다. 

1. AWS 기준으로 설명
 - S3에 파일이 업로드가 될 경우 Lambda가 호출이 되게끔 트리거를 걸어 파일의 무결성 검사
 - Database에 직접적으로 접근이 아닌 Lambda를 통해 간접적으로 접근 하는 방식으로도 사용이 가능 함.
 - 외부 망에서 Lambda를 직접적으로 호출 또한 가능 함.

FaaS는 함수를 호출 할 때만 비용을 청구 하고, 동시에 Lambda를 호출 하더라도 독립성을 가지는 서비스 이다. 즉 함수가 독립적으로 동작을 한다는 의미는 동시에 수백, 수천개의 함수가 호출을 하더라도 성능 하락이 존재 하지 않는다는 점이다.

해당 부분이 가능 하다면 Scale Out 이 정말로 쉽다는 의미이다.
[![Scale Out](/assets/img/post/2021-07-12-scaleout.png)](https://insights.illumio.com/post/102foor/scaling-up-vs-scaling-out-your-security-segmentation)

##### FaaS 제공 해주는 클라우드 서비스 업체 종류
 - Amazon Web Service의 Lambda
 - Google Cloud Platform의 Functions
 - Azure의 Functions

### 그 외 
 - 1. Baas (Backend As A Service)
이미 완성이 되어 있는 서비스를 API 형식으로 제공을 해준다
    - Firebase
    - Amazon Web Service

Discord API, Google Drive API 등등

 - 2. SaaS (Software As A Service)
클라우드 환경에서 운영되는 애플리케이션 서비스를 말한다. 모든 서비스가 클라우드에서 이뤄지며 소프트웨어를 구입해서 PC에 설치하지 않아도 웹에서 소프트웨어를 빌려 쓸 수 있다.

웹에서 제공 하는 서비스, Google Drive, Office 365, Discord 등등


# 후기
처음에 Amazon Web Service의 Lambda만 공부 하려 책을 구입해서 찬찬히 읽어 보았다. 공부를 하기 전에는 IaaS, PaaS와 같은 *aaS에 대해서 알고는 있었지만 해당 내용이 각각 어떠한 내용인지 잘 알지 못하였다.
이번 기회에 공부하면서 알게 된 내용을 정리를 해 보았고, FaaS가 어떻게 동작하는지 잘 알지 못하였지만 확실하게 알게되는 기회가 되었다.
