---
layout: post
title: GoF Design Patterns 시리즈
description:
img_path: /assets/img
image:
  path: gof-pattern-series-preview.png
  alt: preview image
pin: false
categories: [Architecture, GoF Design Patterns]
tags:
---

**OOP 언어**로 프로그램을 제작해 본 적이 있다면, Class를 설계하고 조립하는 과정에서 많은 시행착오를 겪었을 것이다.
특히 **기능을 확장**하거나 **특정한 로직을 작성**할때 자주 경험하게 된다.

- 애초에 구현에 실패할 수도 있다.
- 구현을 했지만 찜찜함이 남는 경우도 많다.

위와 같은 문제를 해결하기 위해, **GoF Design Pattern**에 대해 알아보자!

# 1. Design Pattern이란?

---

대부분의 프로그래머들은 비슷한 문제상황을 마주하게 된다. 그에 따라 **자주 발생하는 문제들에 대한 해결책**들을 정리해놓은 메뉴얼들이 많이 만들어졌다.

- **Data Structure**는 **데이터 저장구조**에 관한 메뉴얼로 볼 수 있다.
- **Algorithm**은 **데이터 처리**에 관한 메뉴얼로 볼 수 있다.

<br/>

**Design Pattern**은 애플리케이션이나 시스템의 **디자인**에 관한 메뉴얼이다.<br/>
**디자인**이라고만 하면 범위가 너무 넓기 때문에, 관심사에 따라 몇가지로 분류가 되어있다.

- **GoF Design Pattern** : GoF(Gang of Four)에 의해 정리된 패턴이다.
- **Concurrency Pattern** : 동시성 제어에 관한 내용이다.
- **Architecture Pattern** : Application 전반을 관통하는 논리구조에 관한 내용이다.
- **Realtime System Pattern** : 실시간 시스템에 관한 내용이다.
- **etc...** : 그 외에도 몇가지 갈래가 존재한다.

<br/>
이번 시리즈에서는 **GoF Design Pattern** 에 대해 알아 볼 것이다.

# 2. GoF Design Pattern이란?

---

**GoF Design Pattern**은 **언어레벨**에서 **Class의 설계와 조립**에 대한 메뉴얼이다.

# 3. 왜 Design Pattern을 알아야 할까?

---

먼저 말을 하고 가자면, 디자인 패턴은 만능도 아니며 절대불변의 법칙도 아니다. 클래스 다이어그램을 첨부하겠지만 ㅇㅇ은 아니다. 앞으로 이어질 포스팅들은 GoF의 ㅇㅇ 설명할 것이며, 해당 패턴을 관통하고있는 핵심 개념에 집중해주길 바란다. 세세한 구현은 하나의 예시일 뿐이다!

그럼 해당 내용을 명심하고 디자인 패턴의 세계로 뛰어들어보자!

---
