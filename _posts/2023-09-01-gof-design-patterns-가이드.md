---
layout: post
title: GoF Design Patterns 가이드
description:
img_path: "/assets/img"
image:
  path: gof-pattern-series-preview.png
  alt: preview image
pin: false
categories:
- Architecture
- GoF Design Patterns
tags:
date: 2023-09-01 23:33 +0900
---
**OOP 언어**로 프로그램을 제작해 본 적이 있다면, Class를 설계하고 조립하는 과정에서 많은 시행착오를 겪었을 것이다.
특히 **기능을 확장**하거나 **특정한 로직을 작성**할때 자주 경험하게 된다.

- 애초에 구현에 실패할 수도 있다.
- 구현을 했지만 찜찜함이 남는 경우도 많다.

**GoF Design Pattern**을 통해 위와 같은 문제를 어느정도 해결할 수 있다.

## 1. Design Pattern이란?

---

대부분의 프로그래머들은 비슷한 문제상황을 마주하게 된다. 그에 따라 **자주 발생하는 문제들에 대한 해결책**들을 정리해놓은 메뉴얼들이 많이 만들어졌다.

- **Data Structure**는 **데이터 저장방법**에 관한 메뉴얼로 볼 수 있다.
- **Algorithm**은 **데이터 처리**에 관한 메뉴얼로 볼 수 있다.

<br/>

**Design Pattern**은 애플리케이션이나 시스템의 **코드 설계**에 관한 메뉴얼이다.
**설계**이라고만 하면 범위가 너무 넓기 때문에, 관심사에 따라 몇가지로 분류가 되어있다.

- **GoF Design Pattern** : GoF(Gang of Four)에 의해 정리된 패턴이다.
- **Concurrency Pattern** : 동시성 제어에 관한 내용이다.
- **Architecture Pattern** : Application 전반을 관통하는 논리구조에 관한 내용이다.
- **Realtime System Pattern** : 실시간 시스템에 관한 내용이다.
- **etc...** : 그 외에도 다양한 패턴이 존재한다.

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

즉, 프로그래밍의 필수적인 교양이라고 생각할 수 있다.

## 3. 학습 사이트 소개

---

[Refactoring Guru](https://refactoring.guru/ko/design-patterns) 라는 사이트에 굉장히 잘 정리가 되어있다. 해당 사이트를 둘러보는것 만으로도 충분하다고 생각한다.

![Refactoring Guru](refactoring_guru.png)
_Refactoring Guru 사이트_

깔끔한 디자인과 알찬 내용으로 구성되어 있다. 하나의 패턴에 대해서도 다양한 구현방식이 포함되어 있으며, 자체적으로 카탈로그[^fn-nth-1]로 사용할 수 있다.

## 4. 패턴이 절대적인 것은 아니다

---

> **"일반적으로 패턴의 필요성은 개발자가 추상화 수준이 부족한 프로그래밍 언어나 기술을 선택했을 때 발생합니다."** - Refactoring Guru : 패턴에 대한 비판

**Observer 패턴**의 예시를 보자. 카탈로그에 나와있는 대로 코드로 구현하면 어떻게 될까?

**Target**의 **num 필드**를 **Observer**가 감시하려면 다음과 같은 코드구현이 필요하다.

{: file='Target.java'}

```java
// 감시 대상이 되는 클래스
public class Target {
  // observer 들을 관리한다.
  private List<TargetObserver> observerList = new LinkedList<>();

  // 관찰 대상 필드
  private int num;

  // observer 등록
  public void registerObserver(TargetObserver observer) {
    this.observerList.add(observer);
  }

  // observer 삭제
  public void removeObserver(TargetObserver observer) {
    this.observerList.remove(observer);
  }

  // observer에게 알림. setter 등에서 적절하게 호출한다.
  public void notifyObserver() {
    for (TargetObserver observer : observerList) {
      observer.onNumberChange(num);
    }
  }
}
```

{: file='TargetObserver.java'}

```java
// 감시를 하고 싶은 클래스는 해당 인터페이스를 구현해야 한다.
public interface TargetObserver {
  void onNumberChange(int num);
}
```

{: file='Observer.java'}

```java
// 감시를 하는 클래스. 인터페이스를 구현해야 한다.
public class Observer implements TargetObserver{
  ...

  @Override
  public void onNumberChanged(int num) {
    // do something...
  }
}
```

{: file='Main.java'}

```java
// 최종적으로 observer 등록을 해주어야 한다.
...
Target target = new Target();
Observer observer = new Observer();

target.register(observer);
...
```

파일개수가 늘어나고 코드가 지저분해질 뿐더러, 하나의 변수를 감시하기 위해서 무려 4군데의 코드가 변경되어야 한다.

<br/>

**Java 8** 부터는 **Lambda**를 지원한다. Observer는 다음과 같이 간단하게 바뀐다.

{: file='Target.java'}

```java
// 감시 대상 클래스가 변경되는건 거의 동일하다. 하지만 observer 대신 callback을 관리한다.
public class Target {
  private List<Consumer<Integer>> listenerList = new LinkedList<>();

  private int num;

  public void addListener(Consumer<Integer>...callbacks) {
    this.listener.addAll(List.of(callbacks));
  }

  public void removeListener(Consumger<Integer>...callbacks) {
    this.listener.removeAll(List.of(callbacks));
  }

  public void notifyListener() {
    for(Consumer<Integer> listener : listenerList) {
      listener.accept(this.num);
    }
  }
}
```

{: file='Main.java'}

```java
// 간편하게 Lambda 등록만 하면 끝이다.
...
Target target = new Target();
target.addListener(num -> {
  // do something...
});
...
```

Java가 진화함에 따라 훨씬 깔끔한 구현이 가능해진 것을 볼 수가 있다.

이런 것은 GoF 에만 해당되는 이야기가 아니다. **DI 패턴**을 구현한 Google의 [Guice 라이브러리 문서](https://github.com/google/guice/wiki)에는 Java가 마땅히 제공해야 할 기능의 누락을 채워준다고 표현하고 있다. 즉, 다양한 패턴들은 언젠가는 구식이 될 수도 있으며, 절대적인 수칙이 아니다.

{: .prompt-info}

> 또한 위 예제에서 확인할 수 있듯이, 맹신적인 사용은 오히려 코드를 망칠 수 있다.  
> 패턴의 오용 및 남용으로 인해 코드가 오히려 퇴보할 수 있는데, 이렇게 모든 문제를 패턴으로 해결하려는 초기학습단계를 **패턴병**이라고 부른다.

<br/>

**결론적으로 학습과정에서 나오는 Class Diagram 들에 너무 큰 의미를 둘 필요는 없다.**

어차피 구현은 카탈로그마다 조금씩 차이가 있다. 중요한 것은 각각의 패턴들에 담긴 **핵심 아이디어**이니, 세세한 구현내용을 정리하면서 시간을 쓰지 말고 핵심개념들을 마음속에 새기자. 무엇보다 패턴의 적절한 적용에는 이론보다 경험이 훨씬 중요하기 때문에, 여러번 삽질을 해보는 것이 유일한 학습방법이다. 패턴병도 꼭 경험하길 바란다.

<br/>
<hr/>

[^fn-nth-1]: 탄생배경, 솔루션, 장단점, 다른 패턴과의 관계 등 상세한 설명이 포함되어 디자인패턴을 적용하기 전 참고자료로 사용할 수 있다.
