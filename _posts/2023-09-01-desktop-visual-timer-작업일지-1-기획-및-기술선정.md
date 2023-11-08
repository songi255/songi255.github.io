---
layout: post
title: Desktop Visual Timer 작업일지 1 - [기획 및 기술선정]
description:
img_path: "/assets/img/timetimer"
image:
  path: timetimer_unsplash.jpg
  alt: Time Timer - Unsplash의 Ralph Hutter
pin: false
categories: [Project, Desktop Visual Timer]
tags:
date: 2023-09-01 18:46 +0900
---

처음에는 단순한 desktop 용 visual timer가 필요했다. 하지만 직접 만들 생각을 하고나니 부가적인 기능들이 떠올랐다.

- 여러 타이머 **설정값 템플릿**을 지원하면 어떨까?
- 타이머를 설정하지 않고 무계획적인 사용 시 **Notifier** 를 만들면 어떨까?
- **백색소음이나 Lo-fi** 같은 **Music Player** 를 넣으면 어떨까?
- **사용통계**를 볼 수 있는 **Analyzer** 를 넣으면 어떨까?
- 타이머 시간동안 **Site Block** 을 넣으면 어떨까?
- **뽀모도로** 모드를 만들면 어떨까?

초기 구상을 하고 나니 처음부터 볼륨이 너무 커져버렸다.
목표가 너무 막연한 경우 진행이 힘들어진다. 이런 경우 먼저 핵심기능에 집중해서 구현을 해봐야 한다.

프로토타입을 만들어보면 코드에 관한 많은 교훈[^fn-nth-1]을 얻을 수 있다. 다른 기능들은 배제하고 우선 Timer 기능부터 만들어보기로 했다.

## 🛠️ 1. 기술선정

![Tool 사진](tool.jpg)
_<a href="https://unsplash.com/ko/%EC%82%AC%EC%A7%84/IClZBVw5W5A?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>의<a href="https://unsplash.com/ko/@toddquackenbush?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Todd Quackenbush</a>_

먼저 GUI Framework 를 선정해야 했는데, 고려사항은 다음과 같다.

- **Cross Platform(OS)** 지원하는지?
- 직간접적으로 **배경지식**이 있는지? (학습곡선 문제)
- **리소스**를 많이 사용하지는 않는지?
- **라이센스 문제**는 없는지?

<br/>

결과적으로 **JavaFX**를 선택했다. **Qt Designer(C++)**, **electron**등의 선택지도 있었지만 JavaFX를 선택한 이유는 다음과 같다.

- **Qt Designer** 는 라이센스[^fn-nth-2]가 조금 복잡하다. C++도 깊게 공부하지는 않았다.
- **electron**은 chromium 기반이라 다소 무겁다.
- JavaFX는 **사용경험**이 어느정도 있었다.
- **FXML, CSS**를 지원하기 때문에 조금 더 편할[^fn-nth-3] 것 같았다.
- Java를 사용한다면 **Swing**, **AWT**에 비해서는 조금 더 풍부한 UI 구현이 가능하다.

<br/>

**Java version**은 **17**을 사용했는데, 단순히 학습용으로도 17 환경을 경험해보고 싶었기 때문이다.

build 도구는 **Maven**을 사용했다. 단순히 gradle 보다 익숙했기 때문이다.

<br/>

## 🖌️ 2. 디자인

전문분야는 아니지만 평소 UI/UX 글은 챙겨보고 있었다. 비슷한 처지라면 [Google Material Design System](https://m3.material.io/) 을 참고하면 매우 도움이 된다. 참고로 JavaFX에서도 Material Design 구현을 해놓은 [라이브러리](https://github.com/palexdev/MaterialFX)가 존재한다.

### 2-1. UI 디자인

초기 디자인이 굉장히 힘들었는데, 기능이 많고 모호하다보니 화면을 어떻게 구성해야 할 지 감이 잘 오지 않았다. 설정값 변경방식, 반응형 디자인 같은 세부사항 하나하나가 굉장히 괴로웠다. 그래서 어떻게 해결했냐면, 그냥 포기해버렸다.

모든걸 다 잘할 순 없다. 그리고 현재 요구사항은 어차피 변경가능성이 많다. 프로토타입이 완벽할 필요는 없는것이다. 나는 디자이너가 아니니, 꽤 쓸만하다 싶어지면 아는 디자이너 형님에게 부탁해보기로 하고 처음 떠올렸던 핵심 이미지만 담았다.

![Figma 디자인](figma.png){: width="600" }
_Figma 디자인 초안_

그렇게 프로토타입의 초기 이미지가 완성되었다. 노트에 끄적거린 후 **Figma**를 사용하였다.

<br/>

### 2-2. UX 디자인

[Material UX Research](https://material.io/blog/motion-research-container-transform)에 따르면, 컨테이너 전환 애니메이션은 단순히 눈의 즐거움 이상의 중요성을 가진다. 바로 어플리케이션을 사용하는 맥락이 무의식적으로 유지된다는 것이다.

![Context UX](context_ux.gif)
_Container Transition_

그래서 처음부터 위와 같은 **Container Transition**은 필수요소였다. 추가적으로 드래그로 위치 옮기기 외에 다른 세부적인 사항은 경험이 부족하므로 해당 시점에서는 고려하지 않았다.

<br/>
<hr/>

[^fn-nth-1]: 아키텍처 설계, 안티패턴, 예상치 못한 오류 등
[^fn-nth-2]: [Qt 라이센스 목록](https://doc.qt.io/qt-5/licenses-used-in-qt.html). 공식페이지에서도 라이센스 관련 문의는 변호사와 상담하라고 적혀있다.
[^fn-nth-3]: 실제로는 코드측면에서 여러 문제가 많았다. 이후 포스팅에서 작성 예정이다.
