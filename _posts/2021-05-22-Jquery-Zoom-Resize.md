---
layout: post
title: 웹 페이지 외주 개발 중 발생 했던 일 정리
author: piorosen
tags: [웹 페이지, 외주, php, docker, jquery]
hide_title: false
# feature-img: "assets/img/feature-img/"
categories: [Blogging, Develop]
feature-img: "assets/img/post/2021-05-22-dfy_icon.png"
# thumbnail: "assets/img/thumbnails/feature-img/"
# thumbnail: "assets/img/post/2021-05-22-dfy_icon.png"
---

# 프로젝트 소개
2021년 4월 초 부터 2021년 5월 중순 까지 진행이 된 웹 외주를 진행 하였었다. 작년, 2020년 9월 쯤 진행을 하였던 웹 외주를 끝내고 업체 쪽에서 추가적인 개발 요청을 하여 새로운 기능을 구현 및 웹 페이지 재 설계 및 디자인 하게 되었다.

현재는 웹 페이지, 백엔드 개발을 모두 끝내게 되어서 결과물을 링크로 남깁니다. 개발은 누리아이엔에스에서 자동차 보험을 시작하고자 웹 페이지가 필요하게 되었고, 웹 페이지를 개발, 자유롭게 컨트롤이 가능한 관리자 페이지 개발이 필요하게 되었습니다. 

개발 자체는 혼자서 가능 하지만 디자인적인 요소는 많이 부족할 것 같아 디자인과 프론트가 가능한 [황진주(MFDO)](https://github.com/oMFDOo) 와 같이 개발을 하여 마무리를 짓게 되었으며, 개발을 하면서 많은 경험, 새로운 시도, 개발 환경 구축을 하게 되었습니다.

#### 라이브 서버 및 개발 서버
* [현재 웹 사이트](http://directfyou.com)
* [과거 웹 사이트](http://directfyou.com/index-prev.php) 
* [관리자 페이지](http://directfyou.com/manager)<br>
학교 서버 이므로, 미래 추후에 접속이 안될 수 있습니다. 하지만 아래에 이미지로 자세히 표현 합니다.
* [개발 환경 코드 서버](http://test.udon.world:5002)
* [개발 환경 웹 페이지](http://test.udon.world:5003)

----

# 개발 내용
2020년에 진행을 했던 내용은
* 웹 호스팅 서버 선택 및 구입(Cafe24)
* 호스팅 서버 도메인 구입 및 연결(Cafe24)
* [다이렉트 보험 회사 안드로이드 어플 개발](https://play.google.com/store/apps/details?id=com.deu.directfyou)
* 보험 신청 시 문자 알림 서비스 구현
* 보험 신청자, 추천인 목록 다운로드, 보험 회사 목록 추가 기능
* 그 외 관리자 페이지

위의 내용으로 개발이 진행이 되었다. 

2021년 개발 내용은 요약을 하자면
* 서브 도메인 접속 시 서브 도메인별로 다른 화면을 보여 주고 싶음.
* 메인 페이지의 디자인 변경 요청.
* 위의 내용에 따라 추가가 되는 관리자 페이지 추가.

----

2020년에 개발을 진행 하면서 어려운 부분은 아무런 기반이 없어서 대부분 내용을 모두 개발을 하여야 했던 부분이였으며, 지속되는 추가적인 개발 내용이 늘어나게 되면서 여러 다양한걸 추가가 된것 같다.

처음으로 시도 해본 문자 SMS 기능이다. 문자 SMS가 어떻게 [웹 발신] 이라면서 내용이 어떻게 해야지 나오는지 의문 이였었다. 처음에는 전화번호와, 문자 보내는걸 구매 해서 해야 하는가? 라는 고민을 했었는데 그런 고민은 필요가 없었다.

실제 동작은 전화 번호만 개인, 법인이 소유 하고 있는지 인증 절차만 진행을 하고, 문자를 보내는 기능은 Cafe24에서 데이터를 처리 해서 보낸다고 되어 있다. 그래서 전화 인증만 하면 문자 서비스를 구매해서 사용만 하면 되는걸 깨닫게 되었었다.

그 외에 2020년에는 그렇게 백엔드는 구현이 어렵지 않았다. 하지만 프론트엔드는 웹 외주 진행 할 때 대표님이 디자인 수정이 많아 난항을 겪었다.

----

# 개발 환경 구축
2020년에 웹 호스팅 서버를 구입 후, 도메인 만 연결을 한 상태였기에 라이브 서버 이지만, 정식적으로 오픈한 상태가 아니였기 때문에 Visual Studio Code의 Extension 기능 중 하나인 ftp-simple 을 이용하여 개발을 진행 하였었다.
그렇지만 2021년에는 라이브 서버에서 바로 편집이 불가능 하기 때문에 별개의 개발 서버가 필요하게 되었다. 그래서 빠르게 개발을 하기 위해서 Docker 이용하여 Code-Server를 구축하고, apm-docker를 설치 하였다. 
![Docker 상태](/assets/img/post/2021-05-22-docker.png)

----

# 서브 도메인 관련
2021년에 시작한 외주는 서브 도메인으로 접속을 한다면 서브 도메인에 따라서 다른 업체 사이트를 나타내야 했었다.

서브 도메인 관련 해서는 PHP에서 
```php
$_SERVER['SERVER_NAME']
```
를 통해서도메인명, 서브 도메인 명을 파싱 하여 처리 하도록 하였다.

해당 방식으로 구현을 하게 되니 도메인에서 CNAME으로 요청해서 웹 서버에 접속을 하거나, Apache를 통해서 서브 도메인을 인식해서 처리를 하는 방식, 둘 중 하나를 택하여 대표님과 이야기를 해야 했었다.

하지만 Cafe24 에서는 도메인에서 CNAME으로 접속하는 방식을 보안상 이유로 막혀 있었다. 그래서 Apache를 통해서, 즉 웹 서버에서 서브 도메인, 디렉토리를 따로 만들어서 인식 하도록 해야하는 했었다.

새로운 디렉토리 www 란 디렉토리를 만들고 모든 이미지를 www란 디렉토리로 옮기거나, 데이터베이스도 옮기는 작업을 진행 하게 되었었다.

Apache를 통한 방법은 최대 서브 도메인 생성 개수는 30개 이기 때문에 30개 이상의 서브 도메인을 만들게 되는 상황이 온다면, Cafe24에 이야기를 하여 CNAME을 통한 방식을 해야 하는 상황이였다.

그렇지만 대표님과 이야기를 진행 해서 업체가 30개 이상 넘지 않을것 같다 하여 Cafe24와 이야기는 진행 하지 않고 Apache 서버를 이용한 방식으로 택하였다.

----

# 완성된 디자인에 디자인 변경 및 수정 요청
대표님이 절대적으로 "자동차 보험 계산하기 버튼"은 웹 페이지에 들어 갔을 때 보여야 한다고 하셨다. 하지만 해당 내용이 보이기 위해서는 디자인적으로 모두 다 바뀌어야 하는 상황이기도 하였다.

그렇지만 가장 큰 문제는 "모니터" 별로 모두 다 다른 결과를 출력 하고 있었기에 많은 어려움이 있다고 이야기를 하였지만 받아 들여지지 않고 어떻게든 해결 해야하는 상황 이였다.

테스트가 가능한 모니터가 24인치 모니터, 4K 27인치 모니터, 맥북 16인치, 삼성 오디세이, 등등으로 테스트를 진행 해서 모두가 다 알맞은 디자인을 다시 하거나, 화면을 일괄적으로 작게 해서 보이게 할 계획 이였다.

하지만 전체적인 디자인을 건들지 않고 "웹 브라우저"의 높이와 "자동차 보험 계산하기" 버튼의 위치를 계산 하여 화면의 배율을 주는것은 어떠한가? 라는 주제로 접근하게 되었다.

그렇지만 가장 큰 단점으로는 모든 웹 브라우저를 지원 해야 한다는 부분 이였다.

1. Internet Explorer
2. Chrome
3. MS Edge

크게 3개로 테스트 하기로 하였고, 3개가 모두 정상적으로 지원하는 css태그와 웹브라우저 기능을 찾다 보니.
Internet Explorer에서 비 표준으로 지원하는 zoom 을 사용 하려 했지만 Chrome과 IE에서 결과가 달라 포기 하여 다른 내용으로 접근 하였었다.

```css
.ratio {
    width: 100%;
    transform: scale(100);
    transform-origin: 0px 0px;
}
```
와  형식으로 웹 브라우저 내용 배율과, 각각 요소의 위치 조정, 그리고 요소의 위치가 가운데 정렬이 되는데 좌상단 고정을 통해 배율을 정하였다.

그렇지만
```css
.browser {
    overflow: hidden;
    height: XXXvh;
}
```
overflow 은 강제적으로 스크롤을 없애는 내용이지만, width나 height 지정이 되어 있으면 해당 내용 까지 표현을 한다. 그래서 스크롤을 없앤 뒤
vh(vertical height) 뷰 포트(웹 브라우저 높이)로 강제적으로 height를 지정 하였다.
지정하는 방식은 최 하단의 footer 의 위치와 웹 브라우저 높이의 비율을 구해 웹 페이지 줌을 구현 하였다.

아래는 구현을 했을 때, 직접 코드를 작성했던 알고리즘 이다.

```js   
var originOffsetFooter = -1;
var originOffsetCalculator = -1;
var outerHeightFooter = -1;
var outerHeightCalculator = -1;

var ratio = 1;
count = 0;
function resize() {
    count += 1;
    var height = $(window).height(); // WEB Browser HEIGHT
    var width = $(window).width(); // WEB Browser HEIGHT
    if (loaded == 8) {
        $('html').removeClass('no-js');
    }

    if (width > 1251) {
        if (originOffsetFooter == -1) {
            originOffsetFooter = $("#footer").offset().top + $("#footer").outerHeight();
        }
        if (originOffsetCalculator == -1) {
            originOffsetCalculator = $("#calculator").offset().top + $("#calculator").outerHeight();
        }

        var cal = originOffsetCalculator;
        var footer = originOffsetFooter;
        var ss = (height / cal);

        $("#resizeZoom").css("width", (1 / ss * 100) + "%");
        $("#resizeZoom").css("transform", "scale(" + ss + ")");
        $("#resizeZoom").css("transform-origin", "0px 0px");

        $("#heightManager").css("height", ((footer * ss) / height) * 100 + "vh");
        $("#heightManager").css("overflow", "hidden");
        ratio = ss;
    }else if (width <= 1250) { 
        $("#heightManager").css("overflow", "");
        $("#heightManager").css("height", "");

        $("#resizeZoom").css("width", "");
        $("#resizeZoom").css("transform", "");
        $("#resizeZoom").css("transform-origin", "");
    }
}
```

값이 1251px 인 이유는 웹 브라우저 모두 표현이 되는 값이 1250px 이므로, 1251px 이후는 모든 요소가 표시, 디자인적인 변화가 없으므로, 1251px 이후의 값으로 지정 하였다.

그래서 1250px 이하인 브라우저는 줌 기능이 종료, 및 삭제가 되며, 1251px 이상이 된다면 줌이 되어서 표현이 되도록 되어 있다. 

실 사용자가 만약 전체 화면을 사용 한다면 왠만한 경우에는 1251px 가 넘기므로 크게 무관하여 진행 하였다.

----

# 정리
웹 외주를 처음으로 진행 해 보았지만 다행이도 무사히 개발에 성공적으로 한 것 같다. 위에서 다양한 문제를 해결 하게 되었다.

외주 자체는 처음에는 두렵고 엄청 어렵다고 생각했지만 해보면서 교수님의 지원을 받지 않는 연구과제 같은 느낌이 많이 들었었다.

개발 난이도는 연구과제가 더 높은것 같다.