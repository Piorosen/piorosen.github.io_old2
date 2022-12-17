---
layout: post
title: Argo CD와 Github Action을 활용한 GitOps 시스템 구축
author: piorosen
tags: [argocd, cd, github, gitops, k8s, github action]
hide_title: false
categories: [Blogging, Cloud]
---

# 개요
Github Action을 통하여 CI를 구현 하였었다. CI를 구축하면서 자동으로 도커 허브까지 성공적으로 배포 하였지만 실제 서버에 배포하는 과정을 해보지 않았다. 그래서 현재 서버에서 사용 중인 Kubernetes에 CD를 구축 하여 자동 배포 서스템을 구축 하고자 했다. Kubernetes는 [CNCF](https://www.cncf.io/) 재단에서 관리를 하고 있으며, [CNCF 재단에서 관리하는 프로젝트](https://www.cncf.io/projects/)에서 Kubernetes에서 Continuous Delivery(이하 CD) 소프트웨어인 ArgoCD를 선택하게 되었다. 그 외에 다양한 업체를 만나보거나 커뮤니티에 이야기를 나눠보면서 가장 많이 거론이 되면서 많이 쓰이고 있던것이 ArgoCD가 이였기 때문에 ArgoCD를 선택해 보았다.

# 적용

현재 Kubernetes를 SaaS, PaaS 형식으로 만들어주는 소프트웨어를 개발하고 있다. 그 중 Client와 통신하여 로그인 토큰 발급, 계정 관리를 담당하는 서버를 우선 개발하고 있다. 그래서 해당 서버를 개발하면서 로컬로 매번 테스트를 진행 하였다. 하지만 집에서 윈도우, 학교나 외부에서는 맥으로 개발을 진행 하다 보니 외부에 있고 서버가 한 군데에서 관리가 되면 좋겠다는 생각이 들어 CD 작업을 하고자 하였다. 적용을 해 본것은 GitOps이며 코드를 작성하고 빌드, 컨테이너 저장소에 푸시, 실제 서버에 배포가 되는 과정을 구축해 보았다.

먼저 현재 구축한 CI/CD 구조는 개발자가 모든 개발을 완료 한 뒤에 태그를 푸시하면 자동적으로 배포가 되는 구조이다. 그래서 개발자가 v1.0.0 이라고 태그를 푸시하게 되면 Github Action에서 트리거가 발생해 Dockerfile을 빌드하고 Docker Hub에 빌드된 내용을 푸시 한다. 그리고 Docker Hub에 푸시 할 때 태그 정보는 v1.0.0 에서 v를 제외한 1.0.0으로 태그를 붙인다. 여기 까지 CI의 구조이고 CD는 ArgoCD의 경우 특정 레포지토리의 커밋, 태그 정보가 변경이 되면 Sync하여 쿠버네티스에 적용하기 때문에 Github Action에서 CD 작업은 특정 레포지토리에 변경점을 푸시 한다. 그렇게 되면 ArgoCD에서 약 3분 간격으로 Git의 내용을 읽어 변경점을 쿠버네티스에 적용, 배포를 하게 된다.

![argocd](/assets/img/post/2022-03-04-workflow.png)
<br>CI/CD 구조
{: .text-center }


![argocd](/assets/img/post/2022-03-04-argocd.png)
<br>현재 동작하고 있는 ArgoCD
{: .text-center }

# 5가지의 Manifest

> 1. Database의 PVC
> 2. Database의 Deployment
> 3. 외부 통신을 위한 Server-Comm의 NodePort
> 4. Database가 통신을 위한 ClusterIP
> 5. Server-Comm의 Deployment

총 5가지의 Manifest가 있습니다. 그 중 스토리지, 외, 내부망과 통신을 위한 Service, 실제 서버가 중지가 되지 않기 위한 Deployment가 있습니다.

# Github Action의 CI / CD 코드

Github Action에서 CI / CD를 자동화 하기 위해서 사용했던 코드는 아래와 같습니다. 현재 작업을 하고 있는 프로젝트는 [AswCloud](https://github.com/aswcloud)란 Organization이며 [로그인 서버](https://github.com/aswcloud/server-comm)와 [쿠버네티스 통신 서버](https://github.com/aswcloud/server-k8s), [통신 모듈](https://github.com/aswcloud/idl), [iOS 사용자](https://github.com/aswcloud/client-ios), [CD를 위한 레포지토리](https://github.com/aswcloud/argo-cd) 이렇게 프로젝트가 구성이 되어 있습니다.

[![이미지](https://opengraph.githubassets.com/d1c94cbd3f9528f3f94f1eddc5c2e35388f99a05c865544b4e66670aa23122b7/aswcloud/server-comm)](https://github.com/aswcloud/server-comm/blob/main/.github/workflows/docker-image.yml)
<br>CI / CD를 구축한 Github Action 코드
{: .text-center }

# 구축 하면서 생긴 오류

구축은 총 4~6시간 정도 걸렸다. ArgoCD를 구축 하면서 그렇게 많은 시간이 걸린 이유는 ArgoCD가 어떻게 동작하는지 알아야 환경 구축을 어떻게 할지 감을 잡는게 2~3시간 정도 걸렸다. 특히 ArgoCD를 사용 하기 위해서 레포지토리에 k8s의 yaml 파일이 있어야하고 yaml의 값의 변경 유무 확인하는 방식이라는 걸 아는게 오래 걸렸다. (책으로 빠르게 구조를 인식하고 구현 하려고 했지만 블로그 자료 같은 곳에서 자료가 정말 없어서.. 레포지토리를 확인해야 했었다.) 그 외에는 Github Action에서 레포지토리를 푸시 하거나, CD 할 때 yaml을 수정하고 푸시 해주는 프로그램을 찾는것에서 오래 걸렸다. (작업 하고, 테스트 하기 위해서 약 70커밋 정도 했다..) 

# 후기

기존에 서버를 구축해서 사용 할 때 매번 코드를 작성하고, 빌드 한 결과물을 서버에 직접 올려 실행 시켜야 했지만 ArgoCD를 사용하면서 모든 작업이 자동적으로 동작 하니, 순수 코드에만 집중 할 수 있는 환경이 되니 정말 쾌적했다. 정말 너무 좋다.