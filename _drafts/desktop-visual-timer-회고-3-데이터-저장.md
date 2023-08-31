---
layout: post
title: Desktop Visual Timer 회고 3 - [데이터 저장]
description:
img_path: "/assets/img"
image:
  lqip:
  path:
  alt:
pin: false
categories: [Project, Desktop Visual Timer]
tags:
---

이제 프로그램에 데이터를 저장해야 한다.

먼저 데이터 작성 로직이 어디에 어떤 형태로 들어갈까를 생각해보았다.
가장 먼저 떠올릴 수 있는건, storage를 종속성으로 받아서 필요할 때마다 write하는 것이다.

하지만 lombok의 장점을 살릴 수 없게 된다. 또한 코드 중복이 생길 가능성도 있다.

먼저 storage 계층 추상화는 필수적으로 필요하다.

1. 직렬화
2. properties
3. SQLite

뭘 사용하든 api 사양만 맞추면 교체할 수 있다.

프로그램이 작은 프로그램이다 보니 SQLite까지는 필요하지 않다.

그렇게 DataManager를 만들었다.

Template model을 구상했었는데, 이를 일단 반영은 시켰다.

Memento, 혹은 Proxy 를 쓸 생각이 있었다. 하지만 이미 Property Class가 구현이 그런식이다.

DataManager 를 사용해보니 중복되는 코드가 발생했다.

- 값 없을경우 기본 초기값이라던가.
- 단순히 setting 한다던가
- key 생성이라던가. 아무리 helper method를 써도 귀찮기는 마찬가지였다.

앞서 Proxy 구상에서 조금 더 가서 persist를 구현해보고 싶었다. 그러니까... 최종 코드 형태는 다음과 같았으면 했다.

코드 ~~~~

이유는, field 선언에 초기값까지 명시함으로써, 데이터 관련 부분을 한눈에 볼 수 있고, 신경쓰지 않아도 된다.

구현은 현재 field가 primitive타입이기 때문에, setter에 Interceptor를 다는 방식으로 진행해보았다.
(사실 후에 말할거지만, 만약 field를 property 클래스로 설정했다면 changeListner를 다는 방식으로 해결할 수 있었다...)

이를 위해 AOP를 사용할 수 있는데, Guice 도입시에 AOP 지원여부를 확인했었기 때문에, Guice 가이드를 보면서 실행해보자.
필드에 주석이 달려있기 때문에 setter 컨벤션을 활용한 방식을 사용하였다.
주의할 점은, 실제 실행되는 곳은 Guice가 만든 Proxy 객체에서 실행되기 때문에, 한꺼풀을 벗겨주는 작업이 필요하다.

성능은 지금 시기에서는 걱정하지 않아도 된다.

1. 저장 데이터의 양이 많지도 않으며
2. 데이터 저장이 그렇게 자주 일어나지 않으며
3. lazy저장, 캐시 등 최적화방법이 당장 떠오르는 것이 많기 때문이다. (property 클래스는 Map 형식이라 오버헤드가 크지 않다.)

이제 잘 작동하는 것을 볼 수 있다.

이를 @Bean 에만 스캔되게 해놨는데, 이건 문제의 소지가 있다. @Store로 바꾸던지 했어야 했다.
스포일러를 하자면, AOP 자체가 refactoring 대상이다.

데이터 저장 장소는 OS에 따라 다르다.
window, linux 등...
