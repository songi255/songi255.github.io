---
layout: post
title: GoF Design Patterns 가이드
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

위와 같은 문제를 해결하기 위해 **GoF Design Pattern**에 대해 공부해보자!<br/>

## 1. Design Pattern이란?

---

대부분의 프로그래머들은 비슷한 문제상황을 마주하게 된다. 그에 따라 **자주 발생하는 문제들에 대한 해결책**들을 정리해놓은 메뉴얼들이 많이 만들어졌다.

- **Data Structure**는 **데이터 저장구조**에 관한 메뉴얼로 볼 수 있다.
- **Algorithm**은 **데이터 처리**에 관한 메뉴얼로 볼 수 있다.

<br/>

**Design Pattern**은 애플리케이션이나 시스템의 **디자인**에 관한 메뉴얼이다.
**디자인**이라고만 하면 범위가 너무 넓기 때문에, 관심사에 따라 몇가지로 분류가 되어있다.

- **GoF Design Pattern** : GoF(Gang of Four)에 의해 정리된 패턴이다.
- **Concurrency Pattern** : 동시성 제어에 관한 내용이다.
- **Architecture Pattern** : Application 전반을 관통하는 논리구조에 관한 내용이다.
- **Realtime System Pattern** : 실시간 시스템에 관한 내용이다.
- **etc...** : 그 외에도 몇가지 갈래가 존재한다.

<br/>
**Design Pattern**이라고 하면 대부분 **GoF Design Pattern**을 의미하는데, 가장 유명하기도 하고 제일 기본이 되는 내용이기 때문이다.

## 2. Design Pattern을 알아야 하는 이유

---

Design Pattern을 알면 문제해결에 큰 도움이 되지만, 사실 더욱 중요한 이유가 있다.<br/>
바로 **의사소통**이다.

- **동료와 협업**할 때, **동일한 기본개념(맥락)**을 알고 있는 것은 의사소통에 큰 이점을 가져다준다.<br/>
- **다양한 기술문서**에서도 **Proxy**등의 개념을 심심찮게 찾아볼 수 있다. 이해력을 높이기 위해서라도 필요한 개념인 것이다.

**GoF Design Pattern** 자체는 **프로그래밍 언어 레벨**에서 **Class의 조립과 설계**에 집중을 한다.<br/>
하지만, 그 **기저에 깔린 개념**은 프로그래밍 언어에만 국한되지 않는다.<br/>

즉, 프로그래밍 세계에서의 필수적인 교양이라고 생각할 수 있다.

## 3. 학습 가이드

---

### 3-1. 언제 사용할 지 생각할 것

### 3-2. 구현에 매몰되지 말 것

공부를 하다보면 Class Diagram 들이 많이 나올텐데, 여기에 매몰되지 말자. 구현은 책마다 다 다르다.

중요한 것은 개념이다. 문제상황에 봉착했을 때, 이런 개념이 있었다는 것을 떠올릴 수 있어야 한다는 뜻이다.

[Refactoring Guru](https://refactoring.guru/ko/design-patterns)

그래서 코딩을 조금 해봤다면 대부분의 개념을 이미 알고있을 것이다.

- **다형성**을 활용해 보았다면 이미 **Strategy Pattern(전략패턴)**을 알고있다!
- **Listner API**를 사용해 본 적이 있다면 이미 **Observer Pattern(옵저버 패턴)**을 알고있다!

이처럼, GoF Pattern은 크게 특별한 개념이 아니다.

## 4. 패턴병
