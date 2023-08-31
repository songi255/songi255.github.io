---
layout: post
title: Desktop Visual Timer 회고 2 - [기초 설계]
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

우선 진행과정에 특수성이 있다.

1. 다른 일을 하면서 사이드 프로젝트가 가능할까? 를 시험해보고 싶었다.
2. 혼자 진행했다.
3. 기획이 명확하지 않다. 언제든지 수정될 수 있다.
4. 소스는 공개해서, 마음에 안드는 부분을 뜯어보게 하고 싶었다.(오픈소스) 아이템 자체도 나쁘지 않다고 생각했기에... 혹시 나중에 함께 하고자 하는 사람이 있다면 같이 하고 싶었다.

이 때문에 몇가지 전략이 취해졌다.

git은 따로 branching 하지 않았다. 적어도 프로토타입 이후에 내놓을 것이었기 때문에.
component 형식 개발을 의도했다. 이건 내가 React를 사용했던 경험때문이기도 하다.

검색결과, fxml에서 include 를 지원하는 것을 확인하였고, 그렇게 시작했다.

## 1. 아키텍처 분석

우선 JavaFX 프레임워크의 아키텍처를 가볍게라도 보고 갈 필요가 있다.

JavaFX는 MVVM 패턴을 사용하는데 대략적인 설명은 ~~와 같다.

처음부터 완벽한 아키텍처가 나오기는 힘들다.
아무리 경력있는 아키텍트라도 ~~~~. DDD 책에서 나온 말이다.
심지어 내가 경력이 있는것도 아니고, 이런 경우에는 역시 둘 사이에서 잘 조율해서 진행해야 한다. 부딧혀봐야한다.
프로토타입을 먼저 만들도록 하자.

다만, MVVM패턴이 기반이므로, 기반이 되는 Model을 잘 짜놓으면 기획 등이 변경이 되어도 큰 타격은 면할 것이다.

## 2. Model 설계

확정된 기획은 아니지만, 대표적인 요구사항을 다시 떠올려보자.

1.
2.
3.
4.

그러면 Timer, Template, Notifier, Player 등등... 다양한 모델들이 떠오른다.
하지만 요구사항이 명확하지 않은 지금같은 시점에서는 모든 모델의 협력을 다 설계할 수는 없다. (기획이 바뀌면 언제 무용지물이 될지 모르니까.)

이런 경우, 변할 가능성이 가장 적은 핵심기능인 Timer 모델부터 만들어보는 것이 합리적이다.
기본적인 props를 정의하고, observer를 달아주자! 다양한 변화에 대응할 수 있다.

이제 다른 Model 확장 시 문제가 없을 지를 한번 생각해보면, 큰 문제는 예상되지 않는다!

observer 구현은 몇가지 디자인이 있었다.

1. Listner Interface
2. callback
3. JavaFX 기본 제공 Property객체 사용하기

기존에는 1번방식으로 했는데,
생각보다 listner 달 일이 많았고
코드가 더러워졌고
변경사항이 많았다.

결과적으로 글을 쓰는 지금 시점에서는 property를 사용하는것이 적절했다고 생각하며, 리팩토링 예정이다.
이유는 생각보다 단순 bind 가 필요한 경우가 꽤 있었고
listner 달기가 쉬웠고
javafx랑 호환이 잘 되니까. 어차피 JavaFX를 쓰기도 하고, 이 객체가 너무 무거워서 성능에 문제가 될 일은 없을텐데, 잘못 생각했었다.

## 3. Singleton Container

이제 객체간 연결에 대해 생각해보자.
MVVM 모델에서는 Model - ViewModel간 관계가 1:1만 있는 것이 아니다. 엮이고 엮인다. 특히 나같은 Component식 개발에서는 더욱.
그러면 동일 Model 인스턴스를 넣어주는것도 일이다.

모델들은 이제 서로 observer 등을 통해 협력하게 된다.
또한 view 계층들도 다양한 모델들을 가져와서 사용하게 될 것이다.
JavaFX는 어떤 Page가 load될 때마다 Controller가 생성된다. 그때마다 넣어줘야 한다.

이런 경우 Singleton으로 작성하는게 좋다.
하지만, 꼭 모든 객체가 Singleton일 필요는 없다. 코드도 길어질 뿐더러 관리도 귀찮다.
이런 경우 Singleton Container 클래스를 만들어, Model들을 여기서 관리하도록 하면 단 1개의 Singleton 만으로도 제어가 가능하다.

Controller에서는 이 Singleton Container를 얻어서 종속성을 얻는다.

Container 초기화 단계에서는 Model간 종속성을 엮어줘야 하는데, Assembler 라는 클래스를 별도로 분리해서 책임을 전가할 수 있다.

## 4. DI 도입

하지만 Assembler만으로 DI를 제어하는 것은 몇가지 문제가 있었다.
첫번째로 Model이 하나 늘어날 때 마다 적어야 할 코드는 더 많아진다.
또한 설계가 변경되는 경우 계속해서 수정해야 한다.
그리고 순환참조 문제로 setter를 사용해야 한다. 즉, final 을 사용하지 못한다.

결국 DI를 도입하기로 했다. 제일 먼저 떠오른 건 Spring Context였지만 Guice를 선택했다.
이유는 Spring Context는 Spring Portfolio들과의 연동을 염두에 두고 설계된 거라 조금 무거울 것 같아서였고,
Guice또한 구글에서 만든 검증된 라이브러리였고, lifecycle지정, AOP까지 지원을 하는 것을 확인했기 때문이다.
참고로 둘 다 Proxy 방식이고, JSR330을 준수한다.
Guice를 사용해 본 결과, 기능면에서 둘 중 뭘 쓰나 큰 차이는 없었다.
오히려 DB로 바꾸고 싶다면 Spring Data를 도입해서 더 나을수도 있었을 것...

Spring의 사용경험을 토대로 Bean, Component 등을 용도에 맞게 나누었다. (JavaFX Service 이해부족 이슈)

Guice는 Component Scan을 지원하지 않는다. (라이브러리 철학과 관련이 있다.) 그래서 Reflection Module 을 직접 작성해야 했다.

그렇게 Guice를 도입하고 난 후 변경사항을 조금 더 편하게 변경할 수 있었다.
