---
layout: post
title: Desktop Visual Timer 회고
description: desktop visual timer project 소개
img_path: "/assets/img/timetimer"
image:
  lqip:
  path: timetimer_unsplash.jpg
  alt: Time Timer - Unsplash의 Ralph Hutter
pin: false
categories:
  - Project
  - Desktop Visual Timer
tags:
date: 2023-08-31 22:51 +0900
---

오래전부터 **Time Timer** 라는 제품을 사용중인데, 꽤 단순해 보이는 이 타이머는 생각보다 큰 의미를 가지고 있다.

- 인간은 시각적 요소를 처리하는 능력이 굉장히 띄어나기 때문에 글자와 그래프는 큰 차이가 있다. **한번 슬쩍 보는것 만으로도 시간이 얼마나 남았는지 알 수 있기 때문**에 생각보다 큰 변화가 있다.
- 타이머를 맞추기 전에, 해당 시간동안 집중을 해도 별다른 일이 생기지 않는지를 확인한다. 이렇게 함으로써 **걱정이나 잡생각없이 집중**할 수 있다.
- 몰입의 관점에서 **집중을 시작하는 습관**으로의 의미가 있다. 또한 **남은 시간(목표)를 확실히 확인**할 수 있기 때문에, 집중이 흐트려질 때 자신을 다잡을 수 있다.

하지만 타이머를 매번 들고다닐 수는 없어서 Desktop 프로그램이 필요했다. 컴퓨터 작업에 방해가 되지 않아야 했는데, 투명하게 오버레이되는 것이 적절해보였다. 하지만 github에서도 그런 앱은 찾지 못했기에 직접 만들기로 했다.

<br/>

> 아직 프로토타입 버전이긴 하지만 결과물은 아래와 같다.

![demo image](demo.gif)
_focus timer app_

다소 간단해 보이는 이 타이머 프로그램을 만들기 위해 어떤 과정들을 거쳤는지 시리즈를 통해 공유하고자 한다.  
모범사례같은건 아니고, JavaFX를 사용하면서 겪어본 실수와 문제점들을 공유하자는 취지이다.  
소스코드는 [Github](https://github.com/songi255/focus-timer) 에서 확인할 수 있다.
