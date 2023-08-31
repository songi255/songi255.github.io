---
layout: post
title: Desktop Visual Timer 회고 1 - [기획 및 기술선정]
description:
img_path: "/assets/img/timetimer"
image:
  lqip:
  path: timetimer_unsplash.jpg
  alt: Time Timer - Unsplash의 Ralph Hutter
pin: false
categories: [Project, Desktop Visual Timer]
tags:
---

우선, 처음에는 단순한 desktop 용 visual timer가 필요했다.
하지만 막상 만들 생각을 하고나니, 부가적인 기능들이 떠올랐다.

- timer를 설정하지 않고 무계획적인 사용 시 notifier 를 만들면 어떨까?
- white noise 혹은 music player를 넣으면 어떨까?
- analyzer를 넣으면 어떨까?
- site block 기능을 넣으면 어떨까?
- pomodoro mode를 만들면 어떨까?

이런 초기 구상을 하고 나니 큰 문제가 생겼다. 처음부터 볼륨이 너무 커져버린 것이다. 욕심을 부리면 안된다...
이런 경우 먼저 핵심기능에 집중해서 구현을 해봐야 한다. 그래서 다른 기능들은 배제하고 우선 timer 기능부터 만들어보기로 했다.

## Figma 디자인

그렇게 초기 디자인을 Figma로 그렸다.(사진)
나는 디자인에는 깊은 조예가 없어서, 그냥 프로토타입이라고 생각하고 만들었다.

기본적인 마우스 동작은 다음과 같았다. (사진)
이는 처음부터 마음속에 있었던 내용이었다.

UI/UX에 관한 글은 항상 읽고 있는데, 애니메이션은 굉장히 중요한 요소이다.
google material design(사진 첨부)에 따르면, 이는 program 자체의 context를 유지시키는 의미도 있다.
무엇보다,보는 재미가 있다.

## 기술선정

우선 몇가지 조건을 충족해야 했다.

1. cross platform 지원할 것.
2. 학습곡선이 가파르지 않을것. 그저 사이드프로젝트였으므로...
3. 리소스를 많이 잡아먹지 않을 것.

3개정도의 후보가 있었다.

1. Qt
2. Electron
3. JavaFX

이중에서 나는 JavaFX를 선택했다.

1. Qt는 그렇게 많이 써보지 않았다. 그리고 나는 c++을 깊게 파지는 않았다. (검색필요 : 라이센스 문제도 존재했다.)
2. Electron은 Chromium기반이라 리소스를 많이 잡아먹는다. (근데 잘못생각한 듯 하다...)

JavaFX를 선택한 이유는 모든 조건을 충족함과 동시에, 내가 예전에 사용해보았기 떄문이다.
그리고 fxml과 css를 지원하기 떄문에 나의 상황에서는 학습할 내용이 적었다.
아니 적다고 생각했다... 사실 조금 실수였다. 이후 포스팅에서 JavaFX의 문제점을 적어놓았다.

Java version은 17을 사용했는데, 단순히 학습용으로도 17환경을 경험해보고 싶었기 때문이다.
하지만 이 또한 사용자의 입장을 생각하지 못했었다.

build 도구는 Maven을 사용했다. 단순히 gradle 보다 익숙했기 때문이다.
