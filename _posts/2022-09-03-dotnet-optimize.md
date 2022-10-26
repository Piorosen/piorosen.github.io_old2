---
layout: post
title: WinAPI 활용한 MONO GUI 잔상 최적화
author: piorosen
tags: [mono, winapi, dotnet, optimize, wm-paint, winform, transparency, rpi4b, gui, linux]
hide_title: false
categories: [Blogging, Develop]
---

# 서론
임베디드 시스템에 GUI를 올려 제품 테스트를 해야 했습니다. 임베디드는 저렴하고, 간단한 테스트를 할 것이였기 때문에 리눅스 이면서 간편한 RPI4B를 이용 하였습니다. 특히 상용 제품이면서, 가격이 저렴해야 했기 때문에 윈도우를 사용할 수 없었습니다. 그래서 QT와 GTK, 또는 MONO(Winform)를 고려 대상으로 보았습니다. QT는 가격이 너무 비쌌으며, GTK는 LGPL 라이센스를 이용하기 때문에 상업용으론 부적합하였다. 그래서 최종적으로 MONO를 사용하기로 결정 하였으며 MONO를 이용한 Linux GUI 개발을 하기로 하였다.

# 개발 중 겪은 문제

MONO는 .Net Framework를 Linux에서 사용하기 위한 목적이면서 Winform을 실행하기 위한 프레임워크이다. 그래서 공식적으로 Microsoft에서 지원을 하는 것이 아니기 때문에 많이 불안정 하다. 특히 이미지 표현 방식이나, 실행 방식이 윈도우와 달라 오류가 발생 했다. 그 중 가장 대표적으로 발생했던 오류는 2가지 였다.

1. 첫 번째는 [Winform은 Control에 투명한 배경을 지원하지 않는다.](https://docs.microsoft.com/en-us/dotnet/desktop/winforms/controls/how-to-give-your-control-a-transparent-background?view=netframeworkdesktop-4.8)
2. 두 번쨰는 RPI4B는 Winform을 돌리기 위해 성능이 매우 느리다.

## Winform은 투명한 배경을 지원하지 않는다?

Winform에선 "공식적"으로 투명한 배경을 지원하지 않는다. 투명한 배경색을 지원하는 것은 오직 WPF만 지원한다. 투명한 배경은 해당 영역을 색을 칠하지 않는것으로, 처리를 한다. 그렇다면 Alpha색이 존재하는 배경색을 사용한 경우 Blend 하여 색을 합한 뒤 배경색을 표현하는 방식으로 처리를 한다. 이러한 방식을 이용하기 때문에 개발자 입장에서는 투명한 색상과, Alpha가 존재하는 것 처럼 느껴진다. 하지만 진정한 투명색이 아니기 때문에 성능, 렌더링 속도에 악영향을 미친다. 가정용 PC는 성능이 좋기 때문에 렌더링이 느린지 잘 알기 어렵다. 하지만 RPI4B는 성능에 문제가 있어 렌더링이 느리다는 것을 알 수 있다.

특히 Winform은 Control은 CreateWindow를 통해 생성이 된 객체와 같기 때문에 Form -> Panel -> Button과 같이 순차적으로 렌더링이 되는 모습을 볼 수 있다.

# 문제 해결 방법

렌더링이 순차적으로 그려지는 모습이 렉이 걸린것 처럼 보인다. 그렇기 때문에 순차적으로 순차적으로 그려지는것이 아닌, 한번에 모든 UI가 그려지도록 해야 한다. 즉, Winform의 렌더링 방식을 파악해야 한다. Winform은 WinAPI를 통해 구현이 되어 있으며, 쓰기 쉽게 랩핑이 되어 있다. 즉, Override를 통해 WM_PAINT, WM_CREATE와 같은 이벤트를 직접 수신하여, 구현 할 수 있다.

직접 WinAPI를 받는 함수를 구현해도 되지만, C#에서 제공해주는 기능인 OnPaint 함수를 override하여 구현 할 수 있다. 아래의 코드는 Background 이미지와, Label, 버튼이미지를 하나의 WM_Paint에서 처리 하도록 구현 하였다.

```cs
protected override void OnPaint(PaintEventArgs e)
{
    base.OnPaint(e);
    BackgroundImageDraw(e.Graphics);
    UpImageDraw(e.Graphics);
    DownImageDraw(e.Graphics);
    LabelDraw(e.Graphics);
}
```

# 결과

Winform에서는 Label의 Background 기본 값이 투명하기 때문에 이미지 위에 Label을 추가한다면 Label이 맨 마지막에 렌더링이 되므로 그려지기 전엔 아무것도 그려지지 않으므로 느려진 것 처럼 보이게 된다. 이때, Winform이 투명 배경을 지원하지 않기 때문에 화면 전환이 일어 났을 때 UI에 구멍이 생기게 된다. 그렇지만, 해당 구멍이 생기는 UI가 생기지 않도록 WM_PAINT 단에서 모든 것이 그려진 뒤 화면에 나타나게 된다면 문제가 발생하지 않는다. 

이러한 문제는 고사양 PC에서도 발생한다. 그렇지만, 눈에 띄게 생기지 않지만 성능이 느린 RPI4B에서 확연하게 보인다. 

# 결론

저수준 단계에서 최적화 및 레이어 통합 이뤄지기 때문에 시각적인 최적화를 이뤄낼 수 있었다. 연산량은 같지만 중간에 어떻게 보이는지에 따라 체감이 다르다. 임베디드 환경에선 최적화가 무엇보다 중요하다.

