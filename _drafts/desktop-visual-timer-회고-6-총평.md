---
layout: post
title: Desktop Visual Timer 회고 6 - [총평]
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

총평
FXML의 이점이 없다... CSS가 잘 지원되는 것도 아니다. width 설정이 어떤건 안된다. resizable 이 아니거든...
CSS가 잘 안되다보니, layout 등을 위해서 직접 Pane을 확장해야 한다. layout 논리는 따로 들어가기 때문에...
결국 FXML 이 아닌 자체적인 View Code를 작성해야 했다. 실제로 잘 분리가 안된다는 말이다. 이건 문제다.

Component형 개발을 하고 싶었는데, 실제로는 Component가 state 를 들고있어야 하는 경우가 많았다. Model로는 애매한 경우가 있다. 예를 들어, mouse event를 처리하기 위한 지역변수들, 혹은 애매한 설정값들은 Model 을 따로 빼기가 애매하다. ORM을 Model에만 적용하겠다는 초기 설계가 어긋났다. 조금 더 독립적으로 작성했어야 했다.
실제로 React 에서도 이를 위해 Component 내에서 state 를 관리하고, 공유해야 하는 경우 context 등의 전역변수를 사용한다.

학습요소가 적다고 표방하지만, 실제로 퀄리티 있는 프로그램을 만들기 위해서는 아키텍처에 대한 이해가 동반되어야 한다. 심지어 FXML 코드가 어떻게 loading 되는지 까지도 알아야 했다.(DI 및 singleton 때문에... factory 지정 등.) FXML 에서는 특히 이점을 느끼지 못했기 때문에, 사용자 층이 많은 Swing 이나 AWT가 더 적절해 보였다.

그리고 관리가 잘 안된다. 예를 들어 Window tray notification 같은 경우, JavaFX에서는 따로 지원되지 않아 AWT 라이브러리를 사용해야 한다. 이때 AWT는 자체 Thread를 또 따로 돌리기 때문에 성능측면에서 문제가 생길 수도 있다.

리팩토링
현재 상태에서도 기능을 덧붙이는데 큰 문제는 없다. 하지만 더욱 깔끔한 코드를 위해 리팩토링이 필요해 보인다.

1. Model field들을 property로 변경한다.
2. Timer Model 에서 State를 제거한다. isRunning() 으로도 충분하며, 나머지 상태는 다른 field로 부터 도출가능하다.
3. Guice, aopallicance, Reflection 종속성 제거. 꼭 DI가 필요하다면 직접 작성할 것.
4. store 로직을 변경할 것.

state - 컴포넌트를 랜더링할때, state에 의해서만 정해져야 하고, 이전 요소에 의해 정해지면 안된다...
state machine 느낌의 패턴을 가져오게 되면 필연적으로 transition 을 생각하기 때문에 이전 요소에 종속적이게 된다.
