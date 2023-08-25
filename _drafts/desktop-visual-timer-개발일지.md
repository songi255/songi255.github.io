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
  Service 내부적으로 fireEvent라던가, updateProgress 등은 모두 platform.runlater 를 사용한다. service실행은 application thread 만 할 수 있고, 실제 task는 worker thread에서 실행되기 때문이다.
  아키텍쳐도 결국, gui update를 담당하는 application thread 에서 update 할 때, concurrence 를 보장해야 하므로, command pattern 으로 처리될 것...(자세한 내용은 찾아보도록 하자...)
  worker thread는 javafx 내부적으로 worker thread pool 을 만들어서 할당하는 듯 하다. 그래서 기존에 있는 thread pool 을 활용하는 느낌으로 사용하자.

Service BugFix

- Service는 javafx Application Thread 에서 조작해야 함.
- 그렇게 하면 javafx concurrent Thread Pool 에서 실행해줌.
- 그래서 concurrent Thread 에서 실행하면 먹통이었던 것.
- Log가 찍히지 않은 것은 logger binding 문제인듯함.
  - reflections Maven Repository 에서 slf4j 구버전 사용확인...

최적화

총평
솔직히 조금 실망스럽다. hello world 프로그램 하나에도 너무 많은 리소스가 소요된다.
FXML의 이점이 없다... CSS가 잘 지원되는 것도 아니다. width 설정이 어떤건 안된다. resizable 이 아니거든...
CSS가 잘 안되다보니, layout 등을 위해서 직접 Pane을 확장해야 한다. layout 논리는 따로 들어가기 때문에...
결국 FXML 이 아닌 자체적인 View Code를 작성해야 했다. 실제로 잘 분리가 안된다는 말이다. 이건 문제다.

Component형 개발을 하고 싶었는데, 실제로는 Component가 state 를 들고있어야 하는 경우가 많았다. Model로는 애매한 경우가 있다. 예를 들어, mouse event를 처리하기 위한 지역변수들, 혹은 애매한 설정값들은 Model 을 따로 빼기가 애매하다. ORM을 Model에만 적용하겠다는 초기 설계가 어긋났다. 조금 더 독립적으로 작성했어야 했다.
실제로 React 에서도 이를 위해 Component 내에서 state 를 관리하고, 공유해야 하는 경우 context 등의 전역변수를 사용한다.

학습요소가 적다고 표방하지만, 실제로 퀄리티 있는 프로그램을 만들기 위해서는 아키텍처에 대한 이해가 동반되어야 한다. 심지어 FXML 코드가 어떻게 loading 되는지 까지도 알아야 했다.(DI 및 singleton 때문에... factory 지정 등.) FXML 에서는 특히 이점을 느끼지 못했기 때문에, 사용자 층이 많은 Swing 이나 AWT가 더 적절해 보였다.

그리고 관리가 잘 안된다. 예를 들어 Window tray notification 같은 경우, JavaFX에서는 따로 지원되지 않아 AWT 라이브러리를 사용해야 한다. 이때 AWT는 자체 Thread를 또 따로 돌리기 때문에 성능측면에서 문제가 생길 수도 있다.
