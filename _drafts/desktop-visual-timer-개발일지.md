---
layout: post
title: Desktop Visual Timer 개발일지
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

MakeTime 이라는 책을 본 적 있는지??
~~ 한 내용이다.

여튼 이게 없더라.

그래서 만들어보았다.

초기 구상은 ~~ 한 내용이다.(사진추가)
여기에 이런 저런 기획이 덧붙여졌다.

~~여기서 확인 가능하다.

기술은... JavaFX썼다.
QT는 라이센스 문제가...

Agile한 개발을 원했다.

observer를 통해서 분리

Assembler를 사용해서 DI 구현...
Guice나 ~~등을 사용해도 됨. 심지어 Spring Container를 DI용도로만 사용하는 경우도 봤음.

초기 구상도는 대충 ~~한 느낌. 어떻게 될 지는 몰랐다.

Canvas로 눈금 그리기

- rotate 사용하기
- Affine 변환 사용하기
- 격자 간격 알고리즘

Overlay Service 구현

- 내분점사용
- fire Event를 통한 연결?

Service vs Thread

- service는 javafx와 연결을 위함.
- onAction 등의 handler와의 연결을 위한 것이다...
  하지만 외부 라이브러리와 연결할 수도 있고... 난 model의 독립성을 사용하고 싶었다.
  Platform.runlater 가 괜히 있는 것이 아니다. 비용은 들지만, 일단 사용해보고 큰 문제가 생기면 최적화하자.

Service BugFix

- Service는 javafx Application Thread 에서 조작해야 함.
- 그렇게 하면 javafx concurrent Thread Pool 에서 실행해줌.
- 그래서 concurrent Thread 에서 실행하면 먹통이었던 것.
- Log가 찍히지 않은 것은 logger binding 문제인듯함.

최적화
