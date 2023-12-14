---
layout: post
title: "Desktop Visual Timer 프로젝트 회고"
description:
img_path: "/assets/img/timetimer"
image:
  path: "timetimer_unsplash.jpg"
  alt: "Time Timer - Unsplash의 Ralph Hutter"
pin: false
categories:
  - Project
  - Desktop Visual Timer
tags:
  - java
  - javafx
date: 2023-08-31 22:51 +0900
---

오래전부터 **Time Timer** 라는 제품을 사용중인데, 꽤 단순해 보이는 이 타이머는 생각보다 큰 의미를 가지고 있다.

- 인간은 시각적 요소를 처리하는 능력이 굉장히 띄어나기 때문에 글자와 그래프는 큰 차이가 있다. **한번 슬쩍 보는것 만으로도 시간이 얼마나 남았는지 알 수 있기 때문**에 생각보다 큰 변화가 있다.
- 타이머를 맞추기 전에, 해당 시간동안 집중을 해도 별다른 일이 생기지 않는지를 확인한다. 이렇게 함으로써 **걱정이나 잡생각없이 집중**할 수 있다.
- 몰입의 관점에서 **집중을 시작하는 습관**으로의 의미가 있다. 또한 **남은 시간(목표)를 확실히 확인**할 수 있기 때문에, 집중이 흐트려질 때 자신을 다잡을 수 있다.

하지만 타이머를 매번 들고다닐 수는 없어서 Desktop 프로그램이 필요했다. 컴퓨터 작업에 방해가 되지 않아야 했는데, 투명하게 오버레이되는 것이 적절해보였다. 하지만 github에서도 그런 앱은 찾지 못했기에 직접 만들기로 했다.

**결과물은 다음과 같다.**
![release image](release.gif)
_focus timer app_

소스코드는 [Github](https://github.com/songi255/focus-timer/tree/release-0.1.0)에서 확인할 수 있다.

## 🛠️ 1. 프로토타입 제작

처음에는 단순한 desktop 용 visual timer가 필요했다. 하지만 직접 만들 생각을 하고나니 부가적인 기능들이 떠올랐다.

- 여러 타이머 **설정값 템플릿**을 지원하면 어떨까?
- 타이머를 설정하지 않고 무계획적인 사용 시 **Notifier** 를 만들면 어떨까?
- **백색소음이나 Lo-fi** 같은 **Music Player** 를 넣으면 어떨까?
- **사용통계**를 볼 수 있는 **Analyzer** 를 넣으면 어떨까?
- 타이머 시간동안 **Site Block** 을 넣으면 어떨까?
- **뽀모도로** 모드를 만들면 어떨까?

초기 구상을 하고 나니 처음부터 볼륨이 너무 커져버렸다.
목표가 너무 막연한 경우 진행이 힘들어진다. 이런 경우 먼저 핵심기능에 집중해서 구현을 해봐야 한다.

프로토타입을 만들어보면 코드에 관한 많은 교훈을 얻을 수 있다. 다른 기능들은 배제하고 우선 Timer 기능부터 만들어보기로 했다.

**완성된 프로토타입은 다음과 같았다.**

![prototype image](prototype.gif)
_focus timer prototype_

외형적으로는 최종 결과물과 큰 차이가 없지만, 코드는 굉장히 차이가 있다. 너무 별로이지만 프로토타입의 소스코드 또한 [Github](https://github.com/songi255/focus-timer/tree/prototype-v0.1)에서 확인할 수 있다.

### 🤔 1-1. 기술선정

![Tool 사진](tool.jpg)
_<a href="https://unsplash.com/ko/%EC%82%AC%EC%A7%84/IClZBVw5W5A?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>의<a href="https://unsplash.com/ko/@toddquackenbush?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Todd Quackenbush</a>_

먼저 GUI Framework 를 선정해야 했는데, 고려사항은 다음과 같다.

- **Cross Platform(OS)** 지원하는지?
- 직간접적으로 **배경지식**이 있는지? (학습곡선 문제)
- **리소스**를 많이 사용하지는 않는지?
- **라이센스 문제**는 없는지?

<br/>

결과적으로 **JavaFX**를 선택했다. **Qt Designer(C++)**, **electron**등의 선택지도 있었지만 JavaFX를 선택한 이유는 다음과 같다.

- **Qt Designer** 는 라이센스[^fn-nth-1]가 조금 복잡하다. C++도 깊게 공부하지는 않았다.
- **electron**은 chromium 기반이라 다소 무겁다.
- JavaFX는 **사용경험**이 어느정도 있었다.
- **FXML, CSS**를 지원하기 때문에 조금 더 편할 것 같았다. 물론 실제로는 아니었다.
- Java를 사용한다면 **Swing**, **AWT**에 비해서는 조금 더 풍부한 UI 구현이 가능하다.

<br/>

**Java version**은 **17**을 사용했는데, openjfx와 관련한 이슈도 있었고, 배포를 위한 [JPacakage](https://openjdk.org/jeps/392) 툴을 사용하기 위해서는 16+ 버전이 필요했다. 사실 처음에는 단순히 학습용으로 17 환경을 경험해보고 싶었다.

build 도구는 **Maven**을 사용했다. 단순히 gradle 보다 익숙했기 때문이다.

<br/>

### 🖌️ 1-2. 디자인

전문분야는 아니지만 평소 UI/UX 글은 챙겨보고 있었다. 비슷한 처지라면 [Google Material Design System](https://m3.material.io/) 을 참고하면 매우 도움이 된다. 참고로 JavaFX에서도 Material Design 구현을 해놓은 [라이브러리](https://github.com/palexdev/MaterialFX)가 존재한다.

#### 1-2-1. UI 디자인

초기 디자인이 굉장히 힘들었는데, 기능이 많고 모호하다보니 화면을 어떻게 구성해야 할 지 감이 잘 오지 않았다. 설정값 변경방식, 반응형 디자인 같은 세부사항 하나하나가 굉장히 괴로웠다. 그래서 어떻게 해결했냐면, 그냥 포기해버렸다.

모든걸 다 잘할 순 없다. 그리고 현재 요구사항은 어차피 변경가능성이 많다. 프로토타입이 완벽할 필요는 없는것이다. 나는 디자이너가 아니니, 꽤 쓸만하다 싶어지면 아는 디자이너 형님에게 부탁해보기로 하고 처음 떠올렸던 핵심 이미지만 담았다.

![Figma 디자인](figma.png){: width="600" }
_Figma 디자인 초안_

그렇게 프로토타입의 초기 이미지가 완성되었다. 노트에 끄적거린 후 **Figma**를 사용하였다.

<br/>

#### 1-2-2. UX 디자인

[Material UX Research](https://material.io/blog/motion-research-container-transform)에 따르면, 컨테이너 전환 애니메이션은 단순히 눈의 즐거움 이상의 중요성을 가진다. 바로 어플리케이션을 사용하는 맥락이 무의식적으로 유지된다는 것이다.

![Context UX](context_ux.gif)
_Container Transition_

그래서 처음부터 위와 같은 **Container Transition**은 필수요소였다. 추가적으로 드래그로 위치 옮기기 외에 다른 세부적인 사항은 경험이 부족하므로 해당 시점에서는 고려하지 않았다.

<br/>
<hr/>

## 🔄 2. 리팩토링

프로토타입이 완성된 후 얻은 다양한 교훈들을 얻었다. 몇가지만 나열하자면

- FXML은 별로 좋은 방법이 아니다.
- JavaFX가 제공하는 바인딩 라이브러리를 적극 활용해야 한다.
- JavaFX의 Controller 는 MVC의 컨트롤러가 아니다. View로 보는 것이 합당하다.
- JavaFX 기본 제공 API들을 통해 랜더링을 제어하는 것이 현명하다.
  - 예를 들면 `ChangeListner` 혹은 `AnimationTimer` 등. JavaFX Application Therad 와 건전한 연동을 가능하게 한다.
  - 이로 인해 `Platform.runLater()`는 웬만하면 쓸 일이 없다.
- 커스텀 DI 만으로도 충분하다.

이외에도 얻은 교훈들이 많았지만 몇개는 그냥 생략했다. 나머지는 기술리뷰 포스팅에서 다룰 예정이다.

제일 중요한 것은 초기 기획과는 달리 **타이머 기능만으로도 충분히 쓸만했다.** 구현의 범위가 정해졌으니 맘편히 리팩토링 해볼 수 있었다.

## 🚀 3. 최종배포, Reddit, 그리고 코드리뷰

완성된 결과물은 Github Release를 통해 배포했다. 아쉬운 점은 Mac과 Linux용 빌드와 테스트는 해볼 수 없었다.

![github release](github_release.png)
_github release_

JavaFX는 그렇게 인기있는 기술은 아니라서 커뮤니티 규모가 작다는 느낌을 많이 받았다. 하지만 Reddit의 JavaFX 커뮤니티는 비교적 활성화가 되어있어 이 곳에 조언을 구하게 되었다.

![reddit article](reddit_article.png)
_reddit article_

사실 코드에는 그렇게 신경을 쓰지 않았다. 유용한 프로그램을 만드는 것, 그리고 릴리즈에만 집중을 했기 때문이다. 하지만 2주가 지난 시점에서 2개의 진지한 [코드리뷰](https://www.reddit.com/r/JavaFX/comments/187d9s8/i_would_like_some_advice_for_my_overlay_pomodoro/)를 받아볼 수 있게 되었다.

정말 충격이 컸다. 적고 싶은 말이 많지만 요약해서 컬쳐쇼크라고 할 수 있을 것 같다. Reddit의 JavaFX 커뮤니티를 둘러보았을 때, JavaFX를 오랫동안 사용해오신 시니어분들이 대부분이었다. 그런데 처음 글을 쓴 뉴비에게 이 정도로 시간을 써서 호의를 배풀 수 있다니.... 심지어 한분께서는 손수 내 프로젝트를 Kotlin으로 변환한 후 [Github에 게시](https://github.com/PragmaticCoding/FocusTimer)해 주셨다. 물론 이분께서도 오랜 경력을 지니신 분이다. 누군지도 모르는 신입을 위해 이 정도로 손을 잡고 이끌어 줄 수 있을까?

많은 것을 느꼈지만 제일 크게 배운 것은 취미, 커뮤니티, 그리고 후배들을 대하는 **진지한 태도**이다. 이 경험을 하고 나서 평소 커뮤니티에 대해 다소 가볍게 생각하고 있었다는 것을 크게 반성하게 되었다. 그리고 한편으로는 씁쓸했다. 너무나도 가벼워져버린 국내 커뮤니티들에 대한 생각들이 떠올랐기에...

## 💬 4. Github 코드기여

비록 댓글은 2개였지만, Repository Traffic에서 꽤 많은 분들이 관심을 가지고 코드를 봐주신 것을 알 수 있었다. 그리고 나는 MIT 라이센스로 해당 프로젝트를 풀었는데, 얼마 뒤 Github에서 다음과 같은 이메일이 왔다.

![github attributer](github_attributer.png)
_github 2FA required_

깃허브에 코드기여를 한 사람들은 2단계 인증이 강제되며, 설정하지 않으면 계정이 곧 잠긴다고 한다! 처음 이 프로젝트를 시작할 때 내가 가졌던 목적과 진심이 Github에도 통했나보다.

분명 JavaFX는 취업과는 동떨어진 기술이 맞다. 하지만 조금 더 특별한 경험을 할 수 있었고, 생각과 시야를 조금 더 넓힐 수 있었다. 굳이 이 프로젝트를 먼저 시작해서 백수기간이 더 길어졌지만 전혀 후회하지 않는다. 재밌는 경험을 공유할 수 있게 되어 굉장히 기쁘게 생각한다.

<br/>

---

[^fn-nth-1]: [Qt 라이센스 목록](https://doc.qt.io/qt-5/licenses-used-in-qt.html). 공식페이지에서도 라이센스 관련 문의는 변호사와 상담하라고 적혀있다.
