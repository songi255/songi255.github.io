---
layout: post
title: Desktop Visual Timer 작업일지 2 - [기초 설계]
description:
img_path: "/assets/img/timetimer"
image:
  path: timetimer_unsplash.jpg
  alt: Time Timer - Unsplash의 Ralph Hutter
pin: false
categories: [Project, Desktop Visual Timer]
tags:
date: 2023-09-06 17:09 +0900
---

핵심화면은 그려봤으니(그래봤자 1개 뿐이지만) 그걸 토대로 프로토타입을 구현해보면 될 듯 하다. 먼저 기반이 될 베이스 코드들을 작성해보았다.

> 아래 글은 JavaFX기본지식이 필요하며, Spring을 사용해 보신 분들에게는 조금은 당연한 이야기들을 담고있습니다.

## 1. 코드스타일 및 전체구조

### 1-1. Component 형식 개발

디자인이나 기획이 불안정했기에 Component방식 개발을 생각했다. 처음에 FXML을 사용해야겠다고 생각했기 때문에 FXML에서 include를 지원하는지 찾아보았고, 실제로 지원하는 것을 확인할 수 있었다.

코드를 구성한다면 아래와 같이 사용할 수 있다. `fx:id`의 reference는 해당 Component의 Parent(Root)가 할당된다.

{: file='main.fxml'}

```xml
<AnchorPane fx:id="mainPage" ...>
  ...
  <fx:include fx:id="header" source="components/header.fxml" ...>
  <fx:include fx:id="main" source="components/header.fxml" ...>
  ...
</AnchorPane>
```

<br/>

스타일링은 CSS로 하기로 했으므로[^fn-nth-1], class 지원여부와 JavaFX특화 문법은 없는지, Cascade가 제대로 적용되는지 등을 점검해보았다.

`-fx-` 접두사가 붙은 특성이 존재함을 확인했으며, `-` 및 다중클래스도 잘 작동하였고 Cascade도 잘 작동하였다.

{: file='ex.fxml'}

```xml
...
<Button styleClass="btn btn-green"/>
...
```

{: file='ex.css'}

```css
btn {
  -fx-border-color: none;
}

btn-green {
  -fx-background-color: "green";
}
```

### 1-2. 패키지 구조

패키지 구조는 다음과 같은데, 특이할 수도 있는 점은 java 폴더안에 FXML과 CSS가 같이 있다는 것이다.

```bash
├── java
│   ├── components
│   │   ├── headerBar
│   │   └── timerDiskView
│   │      ├── timerDiskView.fxml
│   │      ├── timerDiskView.css
│   │      └── timerDiskViewController.java
│   ├── pages
│   ├── config
│   ├── models
│   └── utils
└── resources
    ├── icons
    └── fonts

```

이는 React 사용경험에서 비롯되었는데, 어차피 ViewModel과 View계층은 꽤 밀접하게 연결되있어서 View가 변하면 ViewModel도 변해야 하는 경우가 잦다.
또한 특정 Component에 문제나 수정사항이 있을경우, 관심사가 한곳에 모여있어 확인하기 훨씬 편하다.

다만 Maven은 java 폴더안의 `.java`파일 외에는 무시해버리므로 `pom.xml`에서 해당 리소스들을 옮기는 처리[^fn-nth-2]를 해주어야 한다.

{: file='pom.xml'}

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.fxml</include >
                <include>**/*.css</include >
            </includes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.*</include >
            </includes>
        </resource>
    </resources>

    ...
</build>
```

{: .prompt-warning}

> 처음 시점에서 FXML, CSS를 사용하기로 결정했다는 것이지, 실제로는 둘 다 사용하지 않고 코드로 제어하는 것을 추천한다. 실제로는 여러 문제점이 있었고 이어지는 포스팅에 작성예정이다.

## 2. 핵심모델만 우선설계

우선 JavaFX 프레임워크는 MVVM 패턴을 사용하는데 이에 대한 설명은 생략한다. 다만 MVVM이므로 근간이 되는 Model은 먼저 독립적으로 개발할 수 있음을 알 수 있다. 그리고 Model이 프로그램 전체의 기반[^fn-nth-3]이 되므로, 이를 먼저 만드는 것이 현명하다.

구현할 기능에는 Timer, Template, Notifier, Music Player, Todos 등이 있으며, 언제든지 변경될 수 있다. 사실 지금 상황에서 Model끼리 복잡하게 얽힌 상황은 Template를 제외하면 딱히 없고, 대부분 독립적이다.

- Notifier는 Timer가 멈춘 시점부터 Scheduling 하면 끝이다.
- Music Player는 Timer가 돌고있을때 작동하면 된다.
- Todos 같은건 애초에 독립적이다.

따라서 Timer에 Observer를 붙여주는 것 만으로 확장문제까지 해결할 수 있다. Template으로 인한 데이터주입문제는 다음 포스팅에서 다룬다.
나머지 모델들은 핵심기능이 아니고, 크게 문제될 것이 예상되지 않아 이 시점에서 고려하지 않았다.

{: .prompt-info}

> JavaFX와 Model은 독립적으로 개발될 수 있지만, 꼭 그래야 한다는 것은 아니다. Model의 필드들은 JavaFX의 Property클래스로 선언하는것이 훨씬 낫다. JavaFX와 통합(바인딩 같은)이 훨씬 쉬워지며 JavaFX가 Event-Driven이기 때문에 Listener를 달 일이 꽤 있다. Property를 사용하면 Listener를 쉽게 달 수 있다.

<br/>

Timer 관련 로직은 일반 Thread대신 JavaFX의 Service를 이용하길 권장[^fn-nth-4]한다. Service의 소스코드를 보면 JavaFX내부에서 worker thread pool을 독자적으로 생성하여 사용하고 있음을 알 수 있다.

![executor](executor.png)
_Service 소스코드 일부. Executor 사용을 확인할 수 있다._

굳이 전용 thread pool[^fn-nth-5]을 사용하고 있는데 따로 생성할 필요는 없다.

<br/>

{: file='TimerService.java'}

```java
public class TimerService extends Service<Void> {
    private final TimerModel timerModel;
    private final static long INTERVAL = (long) (1000.0 / 60.0); // 60fps

    ...

    @Override
    protected Task<Void> createTask() {
        return new Task<>() {
            @Override
            protected Void call() throws Exception {
                long startTime = System.currentTimeMillis();
                double startSec = timerModel.getCurTime();
                double passedTime = 0;

                while (passedTime < startSec) {
                    passedTime = (System.currentTimeMillis() - startTime) / 1000.0;
                    timerModel.setCurTime(startSec - passedTime);

                    Thread.sleep(INTERVAL);
                }

                return null;
            }
        };
    }

    ...
}
```

조금 주의할 점은 옵저버가 꽤 자주 실행된다는 것이다. 위 코드에서 `timerModel.setCurTime()` 호출 시 Observer들의 `onTimerChanged()` 코드도 전부 실행된다. 그리고 Service작업은 Worker Thread에서 동작하기에, Observer들이 UI를 업데이트하기 위해서 `Platform.runLater()`가 필요하다.

<br/>

평소라면 다음과 같이 즉석에서 Lambda를 사용할 수 있다.

{: file='Controller.java'}

```java
Platform.runLater(() -> {
  ...
});

// 혹은
Platform.runLater(this::draw);
```

하지만 위 코드는 전부 Runnable 익명객체를 생성한다는 것을 기억해야한다. 1회성이 아닌, 초당 60회 지속적으로 Runnable 객체를 생성하는 것은 그렇게 작은 일은 아니다. (심지어 Observer가 1개가 아니다.) 따라서 `onTimerChanged()`에 연결된 부분만이라도 따로 Runnable로 빼는 것이 좋아보인다.

{: file='Controller.java'}

```java
private final Runnable task = this::draw;

@Override
public void onTimerChanged(){ Platform.runLater(this.task); }
```

물론 이럴 경우 Platform.runLater()의 작업큐에 들어가는 모든 Runnable이 동일한 객체임을 인지하고 논리를 작성해야 한다.

<br/>

## 3. Singleton Container

이제 객체 간 연결에 대해 생각해보자.
MVVM 모델에서는 Model - ViewModel간 관계가 N:1인 경우가 많다. 특히 지금같은 Component식 개발에서는 더욱 심해진다.

![component](component.png){: width="400" height="600" }
_Timer Model의 값을 현재는 2개의 컴포넌트가 사용한다._

JavaFX도 결국 IoC라서 FXMLLoader가 fxml을 로딩하고 Controller 생성 및 바인딩까지 한다. 그러면 동일 Model 인스턴스를 넣어주는것도 일이다. 심지어 Model이 하나만 있지도 않다.

이런 경우 Singleton을 떠올릴 수 있는데, 몇가지 고려를 해봐야 한다.

- 객체 생명주기 - Model객체가 프로그램 처음부터 끝까지 살아있는 상황이 맞을까? 그리고 그게 메모리에 부담이 되지는 않을까?
- Model 객체가 1개만 존재하는 상황이 맞을까?
- MultiThread issue는 없을까?

우선 Timer 여러개를 동시에 돌릴 계획은 없다. 조심할 스레드는 JavaFX 렌더링 스레드(JavaFX Application Thread)가 있다.
버튼을 클릭등의 핸들러 호출을 렌더링 스레드가 하기 때문이다. 하지만 작업스레드 (Service)는 `isRunning()`과 Application Thread에 작업을 위임하는 `onSucceeded()` 같은 함수를 제공하기에 적절한 제어가 가능하다. 즉, 조금만 신경써서 작성하면 굳이 객체 락을 걸지 않더라도 괜찮다.

![singleton container](singleton_container.png)
_Singleton Container_

Singleton으로 결정된 모델들은 여러 Controller에서 사용한다. 그렇다고 해당 모델들을 전부 Singleton으로 구현하면 코드도 길어질 뿐더러 관리도 귀찮다. 이런 경우 Singleton Container 클래스를 만들어 모델들을 관리하게 할 수 있다.

{: file='Controller.java'}

```java
this.timerModel = Container.getTimerModel();

// 혹은 조금 더 추상화를 했다면
this.timerModel = Container.getInstance(TimerModel.class);
```

Container 초기화 단계에서는 Model간 종속성을 엮어줘야 하는데, Assembler 라는 클래스를 별도로 분리해서 책임을 전가할 수 있다. 모습은 다음과 같을것이다.

{: file='Assembler.java'}

```java
...

public void assemble(){
  TimerModel timerModel = Container.getTimerModel();
  Notifier notifier = Container.getNotifier();
  ...

  // weaving
  timerModel.registerObserver(notifier);
  ...
}

...
```

## 4. DI 도입

하지만 Assembler는 몇가지 문제가 있었다. 우선 귀찮다. 모델이 늘어날때마다 함수 사이즈가 커지고, 설계가 수정될때마다 신경써서 수정해야 한다. 또한 모델간 상호의존하는 경우 setter를 사용이 강제된다. 즉 `final` 을 사용하지 못한다. (사실 이런 경우 디커플링을 하는게 우선이긴 하다.) 그래서 DI를 도입하기로 했다.

### 4-1. 라이브러리 선택

제일 먼저 떠오른 건 Spring Context였지만 Guice를 선택했다. Spring Context는 Spring Portfolio들과의 연동을 염두에 두고 설계된 거라 조금 무거울 것 같았다. 만약 SQLite 같은걸 도입하려고 했다면 Spring을 선택했을 수도 있다.

[Guice](https://github.com/google/guice)는 구글에서 만든 검증된 라이브러리이다. Lifecycle, AOP등을 지원한다. Spring, Guice 둘 다 Proxy 방식이고, JSR330을 준수한다.

### 4-2. Component Scan 구현

![component scan](component_scan.png)
_Component Scan_

Guice는 auto scan을 지원하지 않는다. 라이브러리 철학이 필요한 의존성을 명시해야 한다는 입장이기 때문이다. 꼭 사용하고 싶을 경우 Guice 모듈을 작성해줄 수 있다.

{: file='ComponentScanModule.java'}

```java
@Slf4j
public class ComponentScanModule extends AbstractModule {
    private final String packageName;
    private final Set<Class<? extends Annotation>> bindingAnnotations;

    @SafeVarargs
    public ComponentScanModule(String packageName, final Class<? extends Annotation>... bindingAnnotations) {
        this.packageName = packageName;
        this.bindingAnnotations = new HashSet<>(Arrays.asList(bindingAnnotations));
    }

    @Override
    public void configure() {
        Reflections packageReflections = new Reflections(packageName);
        for(Class<? extends Annotation> annotationClass : bindingAnnotations){
            Set<Class<?>> classes =  packageReflections.getTypesAnnotatedWith(annotationClass);
            for(Class<?> clazz : classes){
                log.info("component scanned : " + clazz.getSimpleName());
                bind(clazz).in(Singleton.class);
            }
        }
    }
}
```

Reflections 라는 라이브러리가 있는데, 위 코드만 봐도 느껴지듯 package 대상 리플렉션이 굉장히 편해진다. 이후 Scan 대상클래스에 부착할 Annotation들을 정의하고, Guice 설정클래스에서 해당모듈을 설치해주면 준비완료다.

{: file='Component.java'}

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Component {}
```

{: file='AppModule.java'}

```java
public class AppModule extends AbstractModule {
    @Override
    protected void configure() {
        install(new ComponentScanModule("com.focustimer.focustimer", Bean.class, Component.class));
    }
}
```

이제 `@Bean`이나 `@Component`등의 어노테이션이 붙은 모델들은 Guice에 의해 Singleton으로 관리되며, 편리하게 주입될 수 있다.

{: file='TimerModel.java'}

```java
// 따로 new로 생성하지 않아도 Guice에서 Singleton으로 생성한다.
@Bean
public class TimerModel {
    ...
}
```

### 4-3. JavaFX Factory 연동

이제 마지막 단계는 Controller가 사용할 모델을 자동 주입하는 것이다. JavaFX는 IoC이기 때문에 JavaFX가 Controller를 직접 생성한다. 하지만 DI는 Guice가 생성한 Proxy에 의해 작동한다. 어떻게 해야할까?

놀랍게도 JavaFX는 이런 상황을 고려하여 Controller Factory를 지정할 수 있게 해놓았다. (이 시점에서 정말 감탄했다.)

{: file='Main.java'}

```java
...

@Override
public void start(Stage stage) throws IOException {
    Injector injector = Guice.createInjector(new AppModule());

    FXMLLoader fxmlLoader = new FXMLLoader();
    fxmlLoader.setControllerFactory(injector::getInstance);

    ...
}
...
```

이렇게 Guice의 기본적인 Injector를 사용해서 Controller를 생성하게 할 수 있다.

결과적으로 다음과 같이 `@Injector`를 사용해서 Controller에 의존성을 주입할 수 있게된다.

{: file='MainController.java'}

```java
@Slf4j
public class MainController implements Initializable {
    private final TimerModel timerModel;
    private final OverlayModel overlayModel;

    ...

    @Inject
    public MainController(TimerModel timerModel, OverlayModel overlayModel) {
        this.timerModel = timerModel;
        this.overlayModel = overlayModel;
        ...
    }

    ...
}
```

Controller는 Singleton일 필요가 없다. Guice는 스캔되지 않은 클래스를 대상으로 `getInstance()`를 호출할 경우 매번 새 객체를 반환한다.
물론 의존은 주입된채로 반환된다. 그래서 Controller는 별도의 Component Scan용 Annotation이 붙지 않았다.

DI를 도입한 후, 코드의 수정이 훨씬 편해짐을 느낄 수 있었다.

<br/>

## References

- https://maven.apache.org/plugins/maven-resources-plugin/examples/include-exclude.html
- https://github.com/google/guice
- https://github.com/google/guice/wiki/Scopes
- https://stackoverflow.com/questions/30313403/auto-scan-for-guice
- https://stackoverflow.com/questions/23471185/how-can-i-use-guice-in-javafx-controllers

<br/>
<hr/>

[^fn-nth-1]: 이 시점에서는 그렇게 생각했지만, 기존 CSS와는 느낌이 많이 다르고 문제가 많아 추천하지 않는다. 코드로 직접 제어하길 권장한다. 예를들어 width 속성에 %를 사용할 수 없다.(!) 물론 관련된 오류메시지는 볼 수 없다.
[^fn-nth-2]: resources 폴더에 있다면 굳이 작성하지 않아도 옮겨준다. Maven은 애초에 Java 코딩 컨벤션에 기반한 프로젝트여서, resouces 폴더는 기본값설정이 되어있다.
[^fn-nth-3]: View와 ViewModel은 Model을 사용하기 위한 껍데기일 뿐이라고 생각하면 된다. Model은 그 껍데기와 독립적이어서 JavaFX와 독립적으로 개발될 수 있다. 이는 만들어진 Model을 그대로 사용해서 CLI 프로그램도 만들 수 있어야 한다는 뜻이다.
[^fn-nth-4]: Service에서 고정 Interval이 아닌 JavaFX내부의 pulse timer를 얻어 worker Thread를 제어하는 방법도 생각해볼 수 있다. JavaFX는 기본적으로는 60FPS를 지향하지만, 환경에따라 유동적으로 달라지기 때문이다. 그리고 기존에 Timer관련 로직과 리소스를 사용중인데 굳이 하나 더 만들어 사용할 필요는 없기 때문이다. 하지만 아직 시도해보지는 않았다.
[^fn-nth-5]: 전용이라함은 thread의 시작과 callback은 Application Thread에서만 실행되게 해서 오용을 방지한다던가, JavaFX에 Event를 발생시킬 수 있다던가, Template가 잘 짜져있다던가 하는 것이다. 물론 이런 사항들을 잘 제어만 해줄 수 있다면 따로 java.util.Timer같은 Thread를 사용한다고 해서 큰 문제는 없다. 공식 Docs에도 나와있는 내용이다.
