---
layout: post
title: On-Premise 환경에서 Harbor와 ArgoCD 연동하기
author: piorosen
categories: [Blogging, Develop]
tags: [harbor, argocd, yaml, bash, secret, kubernetes, k8s, infra, https, http, secure]
hide_title: false
---

# 개요

현재 비용상의 문제로 인해 클라우드 서비스에 대한 고려는 진행 중에 있지 않으므로, 내부 시스템에 대해서 모든 기능들은 온프레미스 환경으로(프라이빗 클라우드와 유사하게) 진행하고 있다. 자연스럽게 온프레미스 환경에서 직접 모든것을 구축하여 사용하여야하는 환경에 놓여져 있기에, 클라우드에서는 편리하게 사용할 수 있는 기능들을 사용할 수 없는 상황이였다. 대표적으로는 `AWS`에 있는 `ECR`과 `ECS`간의, (Registry 서버와 Kubernetes 간의) 데이터 공유에 따로 설정 없이 바로 가능한 부분이 있다. 하지만, 온프레미스 환경에서 직접 구축한다면, 신뢰할 수 없는 저장소로 인식이 되는 문제가 있으며, 보안상의 이유로 `Harbor`에 인증 기능을 도입하게 된다면 더욱 난감해지게 된다.

즉, 우리들은 편리하게 사용하였던 모든 기능들이 번거로운 기능들로 변경이 된다는 의미이다. 필자의 경우에는 온프레미스 환경으로 개발하다 보니, 비교적 많지 않은 자료의 양에 대해서 조금 놀라게 되어 국문 자료로, 다른 사람들이 더 많이 읽어 보았으면 좋겠다는 취지로 작성하게 되었다.

# Harbor와 ArgoCD 란?

`Harbor`와 `ArgoCD`는 모두 `CI/CD` 를 위한 부분이 있다. 특히, `Harbor`는 `Docker Hub`나 `ECR`과 완전히 동일한 기능을 하고 있다. 이는, 컨테이너를 저장하고 관리하고, 보안 취약점 검사를 수행하는 의미이다. 

(솔직하게 이야기를 하자면 `ECR` 이나 `Docker Hub`를 이용한다면 정말로 쉽겠지만, 각각의 비용상에 대한 문제가 발생하기도 하였으며, 무료로 사용하게 될 경우에는 `Private`한 공간을 사용할 수 가 없다는 단점으로 인해 직접 `Container Registry` 를 이용하게 되었다.)

`Docker Hub`이나 `ECR`을 사용해본 경험이 있다면 Harbor는 오픈소스로 만들어진 컨테이너의 이미지 저장소로 생각하면 된다. 오픈소스로 구성이 되어져 있으므로, 라이센스나, 비용적인 문제는 발생하지 않는다는 장점이 있다. 또한, 온프레미스 환경에서의 경우에는 개인의 컴퓨터의 용량만 충분하다면 많은 양의 이미지를 저장할 수 있다는 장점이있다. 단점으로는 물리적으로 컴퓨터에 대해 문제가 발생하였을 떄, 책임의 소재는 본인에게 있다는 점이나, 해킹에 대해서 취약 할 수 있다는 점이다. 

`ArgoCD`의 경우에는 `Kuberentes`의 `API`와 통신하여(with 충분한 권한) 새로운 파드, 네임스페이스, Secret, PV* 에 대한 제어를 수행하는 프로그램이다. `ArgoCD`의 내부 동작 구조는 5 분 간격으로 `Github Repository`나 `SSH`, `FTP`와 같은 스토리지 서버에 접속하여 `Yaml` 파일을 읽어온 뒤, 변경점에 대해 실행하는 프로그램이다. 이때, 변경점에 대한 실행하는 위치는 `Kubernetes`를 대상으로 수행하게 된다. 즉, `ArgoCD`는 특정 주기마다 저장된 파일을 읽어 온 뒤 `Kubernetes API`에 요청하는 단순한 구조이다.

# 도입 과정

모쪼록, ArgoCD를 도입한 상황이라면, 이미 사전에 `Github Release`나, `Docker Hub`를 이용중에 있었을 것이였을거라 생각된다. 그렇기에 이용중에 공간적인 문제나 조금 더 편리한 확장성과 제어권을 얻기 위해 `Harbor`로 바꾸는 것을 마음에 먹었을 것이다.

처음에는 `Harbor`를 이용하면서 겪었던 문제로는 크게 3개의 목적이 있었다.
- `Harbor`와 `ArgoCD` 연동하고 싶다.
- 도메인을 활용하고 싶다
- HTTPS`를 도입하고 싶다.

순차적으로, `Harbor`에 대한 설치는 매우 쉬웠으며, 간단하였다. 필자의 경우에는 `Helm`을 이용하여 설치 자동화를 진행하였고, `values.yaml`에서 여러 값을 수정하여 도입하였다. 또한, 이미 사전에 `IngressClass with NginX`로 이미 구축을 하였기 때문에 `Ingress`를 기반으로 도입할 수 있었다.

간단한 위의 문장으로 `Harbor`설치를 끝마쳤고, 도메인을 적용하였다. 이제, 남은 부분으로는 `ArgoCD`에 대한 연동과 `HTTPS` 도입에 문제가 생겼었다. 먼저, `ArgoCD` 연동의 경우에는 필자는 처음에는 도저히 이해가 되지 않았었따.

아니! 어떻게,,, `Harbor`와 `ArgoCD`를 연동할 수 있을까?, 내가 분명히 알고있는 바로는, `ArgoCD`에는 `Container Registry`서버에 대한 인증과 관련된 설정이 없는데 어떻게 `Image Pull`을 할 수 있을까? 라는 부분이였다.

# 발생 문제

## HTTPS 도입에 생긴 문제

기본적으로 `Harbor`은 HTTPS 를 지원하였고, HTTP는 안전한 접근이지 않으므로 오류가 발생한다. 또한, 기본적으로 `Harbor`은 HTTPS를 제공하지만, 어딘가로 부터 인증받지 못한 TLS 인증서를 사용하므로 신뢰하지 못하는 인증서라고 경고를 한다. 그래서 필자의 경우에는 아주 심플하게 (Let's Encrypt 를 사용하는것이 제일 정석적이겠지만) `Cloudflare`의 프록시 서버를 통해 문제를 해결하였다.

그렇지만 놀랍게도, Cloudflare에서의 가장 큰 단점이 `Maximum body size`가 있다는 것이다. 기본 플랜에서는 `100MB` 제한이 있다는 것이 가장 큰 문제였다.

[[https://developers.cloudflare.com/workers/platform/limits/]](https://developers.cloudflare.com/workers/platform/limits/)

이유로는 작은 `Container의 Image`라면 문제가 없지만, 파이썬이나 신경망 모델이 포함된 컨테이너의 경우에는 레이어 마다 엄청나게 큰 사이즈를 가지게 되기 떄문에, `Harbor`에 업로드가 되지 않는 문제가 있다.

그래서 해결책으로는 `Cloudflare`를 사용하지 않고, `DNS Only` 로 라우팅한 다음, `Let's Encrypt`나, 차라리 인증된 인증서를 사용하지 않도록 하였다.

## Error of Image Pull

심플하다! `Harbor`와 `ArgoCD` 간 연동하기 위해서 중간에 `Kubernetes`에 `kubernetes.io/dockerconfigjson` 타입의 `Secret`을 생성해줘야 한다.

그렇지 않는다면 `Harbor`와 `Kubernetes`는 연동되지 않고, 신뢰하지 않는 이미지 저장소 이므로 접근 거부를 해버린다. 또한, 알다싶이 `Harbor`에 로그인을 수행하여야하는데 아직, `ArgoCD`에 우리의 `Harbor` 계정 정보를 기입하지 않았으므로 당연하게도(?) 접근이 되지 않는것이 맞다.

그래서 아래와 같이 Secret을 생성하여야 한다.

```yaml
apiVersion: v1
kind: Secret
data:
  .dockerconfigjson: eyJhdXRocyI6eyJoYXJib3IuZXVuamFlLnNob3AiOnsiYXV0aCI6ImFXUTZjSGMifX19
metadata:
  namespace: default
  name: regcred
type: kubernetes.io/dockerconfigjson
```

이때, `.dockerconfigjson`의 값을 `base64`로 디코딩해서 확인해보게 된다면 

`{"auths":{"harbor.eunjae.shop":{"auth":"aWQ6cHc"}}}` 으로 나타나게 되고, 이때, `harbor.eunjae.shop`의 값을 IP 주소나, 도메인 값을 기입하면 된다. 그리고 `auth`에 있는 값을 확인해 본다면, `id:pw` 란 값이 나오게 되는데, 이때 `id`와 `pw`는 `Harbor`의 계정 정보를 기입하면 된다.

이때, Secret의 적용 범위는 `metadata`에 기록된 `namespace`에만 적용이 된다.
