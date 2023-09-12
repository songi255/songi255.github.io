---
layout: post
title: Desktop Visual Timer 작업일지 3 - [데이터 저장]
description:
img_path: "/assets/img/timetimer"
image:
  lqip:
  path: timetimer_unsplash.jpg
  alt: Time Timer - Unsplash의 Ralph Hutter
pin: false
categories:
- Project
- Desktop Visual Timer
tags:
date: 2023-09-12 19:54 +0900
---
이제 프로그램에 데이터를 저장해야 했다. 데이터 저장과 같은 횡단관심사는 별도의 레이어로 분리하는 것이 좋다. 레이어가 분리되면 데이터 저장로직이 변경되어도 Model 이나 View Model 등 다른 계층에서 코드를 변경할 일이 없거나 적어진다. 또한 데이터 저장관련 중복코드를 줄일 수 있다.

개인적인 경험에 의해, 최종형태는 다음과 같기를 바랬다.

```java
public class TimerModel {
    private TimerState state;
    @SaveWithTemplate("Study") private String goalStr;
    @SaveWithTemplate("3600") private double maxTime;
    @SaveWithTemplate("1500") private double startTime;
    private double curTime;

    @AfterDataInject
    public void afterLoadData(){
      ...
    }

    ...
}
```

`@Save` 관련 어노테이션이 붙으면 값 변경시 자동저장이 된다. 템플릿 기능의 경우 저장계층과 더 밀접하게 관련이 있다고 판단하여 `@SaveWithTemplate`안에 녹였다.

장단점이 있는데, 장점은

- 초기값 지정이 데이터를 실제로 사용하는 곳에 위치해 직관적이다.
- 저장되는 필드가 무엇인지 한눈에 확인할 수 있다.

단점은

- 코드 구조가 경직된다. 현재 요구사항에는 편하지만, 변경을 수용할 수 없을 경우 리팩터링이 필요하다.
- 직관적으로 이해는 할 수 있지만, 주입방식을 정확하게 알아야 디버깅이 가능하다.
- 기반 저장로직과 혼용할 수도 있지만, 그렇게하면 혼란이 생길 수 있다.

**결론부터 말하자면 해당 형태를 통한 개발과정은 굉장히 만족스러웠다!** 큰 규모까지는 조금 더 개선이 필요하지만 작은 규모의 앱에서는 생각한 장점이 정말 잘 두드러졌고, 너무 편했다.

## 1. 데이터 저장형태

우선 데이터를 저장하기 위해서 직렬화 방식을 생각해야 하는데, 다음과 같은 선택지가 있다.

1. Java Serializable
2. Java Properties
3. SQLite

우선 Java 직렬화는 워낙 문제가 많아서 제외했다. 또한 프로그램 규모가 작다보니 SQLite 까지는 필요하지 않았다.
현재 상황에서는 간단한 세팅값만 저장하면 됬기에 Java Properties 형태를 통해 key - value 형식으로 저장하는 것이 적절해보인다. Analytics 등의 기능이 추가되면 위 코드 형식이 아니라 csv 등으로 저장하는 유틸리티를 작성하는게 맞아보인다.

## 2. 저장로직 설계

먼저 전체적인 구조는 다음과 같다.

![Data Layer](datalayer.png)
_Data Layer_

데이터 레이어의 구성은 위와 같이 구성하였다.

- DataManager : Property 관리 및 I/O 담당
- DataInjector : Reflection을 통한 초기 데이터 주입(로딩) 담당
- PersistenceProvider : AOP를 통한 데이터 저장 담당

위 구성을 통해 Bean Object는 저장 로직을 포함하지 않고 독립적으로 존재할 수 있다.

### 2-1. DataManager

먼저 Property Map을 관리하고 I/O를 담당하는 DataManager를 작성한다.

```java
@Slf4j
@Component
public class DataManager {
    private final Properties properties = new Properties();

    public DataManager() {
        try (InputStream is = new FileInputStream(getConfigFile())) {
            properties.load(is);
        } catch (IOException e) {
            log.error("config.properties not found.");
            throw new RuntimeException(e);
        }
    }

    public String getData(String key){
        return properties.getProperty(key);
    }

    public void setData(String key, String value){
        String oldValue = getData(key);
        if (value.equals(oldValue)) return;

        properties.setProperty(key, value);
        saveToFile();

        log.info("saved data with key : " + key + ", old value : " + oldValue + ", to new value : " + value);
    }

    private void saveToFile(){
        try (OutputStream os = new FileOutputStream(getConfigFile())) {
            properties.store(os, "saved properties");
        } catch (IOException e) {
            log.error("config.properties file not found.");
            throw new RuntimeException(e);
        }
    }

    private File getConfigFile() throws IOException {
        File file = new File(PathProvider.getAppDataPath() + FileSystems.getDefault().getSeparator() + "config.properties");
        File folder = file.getParentFile();

        if (!folder.exists()) folder.mkdirs();
        if (!file.exists()) file.createNewFile();

        return file;
    }

    public static String generateKey(int templateNum, String... keys){
        StringBuilder sb = new StringBuilder("template-");
        sb.append(templateNum);
        for(String key : keys){
            sb.append('.').append(key);
        }
        return sb.toString();
    }
}
```

외부에 `getData(key)`, `setData(key, value)`, `generateKey(templateNum, keys)` 정도의 기능을 제공한다.

유틸리티로 PathProvider를 사용하는데, OS에 따라서 일반적인 파일저장 위치가 달라지기 때문이다. 참고로 패키징된 jar 파일 내부의 파일은 readOnly라 write를 할 수 없다.

```java
public class PathProvider {
    public static String getAppDataPath(){
        String osName = System.getProperty("os.name").toUpperCase();
        String appDataPath = null;
        String folderPath = null;

        if (osName.contains("WINDOWS")){
            appDataPath = System.getenv("APPDATA");
            folderPath = "FocusTimer";
        } else if (osName.contains("LINUX") || osName.contains("MAC")) {
            appDataPath = System.getProperty("user.home");
            folderPath = ".FocusTimer";
        } else {
            return "unsupported OS";
        }

        return appDataPath + FileSystems.getDefault().getSeparator() + folderPath;
    }
}
```

OS에 저장된 환경변수를 사용하며, 일반적인 저장위치 관행은 위와 같다.

### 2-2. DataInjector

다음으로 실제 Model에 데이터를 주입하는 클래스이다. 외부객체가 Reflection을 통해 주입하도록 함으로써 Model은 독립적으로 존재할 수 있다.

```java
@Slf4j
@Component
public class DataInjector {
    private final Injector injector;
    private final DataManager dataManager;
    private final TemplateModel templateModel;

    @Inject
    public DataInjector(Injector injector, DataManager dataManager, TemplateModel templateModel) {
        this.injector = injector;
        this.dataManager = dataManager;
        this.templateModel = templateModel;

        this.templateModel.addTemplateNumListener(this::injectAll);
    }

    public void injectAll(){
        Set<Key<?>> beanKeys = injector.getBindings().keySet();

        for(var key : beanKeys){
            // Guice 종속적인 코드.
            Object target = injector.getInstance(key);
            Class<?> targetClass = target.getClass().getSuperclass();

            if (targetClass.isAnnotationPresent(Bean.class)) {
                Method[] methods = targetClass.getMethods();
                Method beforeCall = null;
                Method afterCall = null;

                for(var method : methods){
                    if (method.isAnnotationPresent(BeforeDataInject.class)) beforeCall = method;
                    if (method.isAnnotationPresent(AfterDataInject.class)) afterCall = method;
                }

                try {
                    if (beforeCall != null) beforeCall.invoke(target);
                    inject(target);
                    if (afterCall != null) afterCall.invoke(target);
                } catch (IllegalAccessException | InvocationTargetException e) {
                    throw new RuntimeException("method invocation error", e);
                }
            }
        }
    }

    public void inject(Object targetObj){
        Class<?> clazz = targetObj.getClass().getSuperclass();
        Field[] fields = clazz.getDeclaredFields();

        for(Field field : fields){
            Save save = field.getAnnotation(Save.class);
            SaveWithTemplate saveWithTemplate = field.getAnnotation(SaveWithTemplate.class);
            if (save == null && saveWithTemplate == null) continue;

            String key = clazz.getSimpleName() + "." +  field.getName();
            if (saveWithTemplate != null) key = DataManager.generateKey(templateModel.getTemplateNum(), key);

            String defaultValue = save == null ? saveWithTemplate.value() : save.value();

            String fieldName = field.getName();
            String setterName = "set" +
                    Character.toUpperCase(fieldName.charAt(0)) +
                    fieldName.substring(1);

            try {
                Class<?> fieldType = getWrapperClass(field.getType());
                String fetchedValue = dataManager.getData(key);
                if (fetchedValue == null) fetchedValue = defaultValue;

                Method setter = clazz.getMethod(setterName, field.getType());
                setter.invoke(targetObj, fieldType.getConstructor(String.class).newInstance(fetchedValue));
            } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
                throw new RuntimeException("setter not exists on " + clazz.getName(), e);
            } catch (InstantiationException e) {
                throw new RuntimeException("field type is not supported", e);
            }
        }
    }

    private Class<?> getWrapperClass(Class<?> clazz) {
        if (clazz.isPrimitive()){
            if (clazz.equals(boolean.class)) return Boolean.class;
            if (clazz.equals(byte.class)) return Byte.class;
            if (clazz.equals(short.class)) return Short.class;
            if (clazz.equals(char.class)) return Character.class;
            if (clazz.equals(int.class)) return Integer.class;
            if (clazz.equals(long.class)) return Long.class;
            if (clazz.equals(float.class)) return Float.class;
            if (clazz.equals(double.class)) return Double.class;
        }
        return clazz;
    }
}

```

외부에 `inject(target)`와 `injectAll()`을 열어둠으로써 원하는 시점에 직접 inject 할 수도 있게 하였다.

inject 과정은 target 오브젝트의 모든 필드에 어노테이션여부를 검사한 후 적절한 setter를 네이밍컨벤션을 통해 호출한다. setter 호출 전후에 `@BeforeDataInject`나 `@AfterDataInject`가 붙은 메서드가 존재하면 순서에 맞게 호출한다.

이는 매번 모든 필드를 검사하게 되므로 비효율적인 부분이 존재한다. 하지만 Inject는 초기화때나 가끔 발생하는 이벤트이고, 캐싱이나 바이트코드조작을 통한 최적화가능성이 확실하므로 큰 성능문제가 생기기전까지는 놔두도록 한다.

`injectAll`의 경우 Guice가 관리하는 프록시들을 대상으로 호출하도록 작성하였기에, 프록시를 한꺼풀 벗겨주는 등의 동작이 Guice에 종속적이게 작성되었다.

### 2-3. PersistenceProvider

마지막으로, Setter 호출 시 자동저장되게 하였다. 몇가지 방법이 존재하는데,

- 자체 Proxy 사용
- Observer 달기
- Guice의 AOP 사용

등이 있다.

자체 Proxy는 구현이 귀찮다. 사실 javafx.Property 클래스를 사용하여 Observer를 다는것이 제일 좋은 방법이었다고 지금은 생각한다. 하지만 이 시점에서는 Model은 독립적이어야한다는 강박이 있었고, 필드를 모두 primitive type으로 선언해버리는 바람에 AOP가 가장 편한 방법이었다.

```java
@Slf4j
public class PersistenceProvider implements MethodInterceptor {
    private final boolean saveWithTemplate;
    private DataManager dataManager;
    private TemplateModel templateModel;

    public PersistenceProvider(boolean saveWithTemplate) {
        this.saveWithTemplate = saveWithTemplate;
    }

    @Inject
    public void setDataManager(DataManager dataManager) {
        this.dataManager = dataManager;
    }

    @Inject
    public void setTemplateModel(TemplateModel templateModel) {
        this.templateModel = templateModel;
    }

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("setter " + invocation.getMethod().getName() + " intercepted by Persistence Provider");

        Class<?> targetClass = invocation.getThis().getClass();
        String className = targetClass.getSimpleName().split("[$]")[0]; // Guice 종속적 코드
        String methodName = invocation.getMethod().getName();
        String fieldName = Character.toLowerCase(methodName.charAt(3)) + methodName.substring(4);

        String key = className + "." + fieldName;
        if (saveWithTemplate){
            key = DataManager.generateKey(templateModel.getTemplateNum(), key);
        }
        dataManager.setData(key, String.valueOf(invocation.getArguments()[0]));

        return invocation.proceed();
    }
}
```

위 코드는 Guice의 AOP사용코드인데, 아래 등록절차를 통해 특정 메서드들의 호출 전 `invoke`가 실행된다.

```java
public class AppModule extends AbstractModule {
    @Override
    protected void configure() {
        install(new ComponentScanModule("com.focustimer.focustimer", Bean.class, ServiceBean.class, Component.class));

        PersistenceProvider persistenceProvider = new PersistenceProvider(false);
        PersistenceProvider persistenceProviderWithTemplate = new PersistenceProvider(true);
        requestInjection(persistenceProvider);
        requestInjection(persistenceProviderWithTemplate);
        bindInterceptor(Matchers.annotatedWith(Bean.class), getSetterMatchers(Save.class), persistenceProvider);
        bindInterceptor(Matchers.annotatedWith(Bean.class), getSetterMatchers(SaveWithTemplate.class), persistenceProviderWithTemplate);
    }

    @SafeVarargs
    private Matcher<Method> getSetterMatchers(final Class<? extends Annotation> ...annotationClasses) {
        return new Matcher<Method>() {
            /**
             * Validates
             *  - method is setter.
             *  - field with setter exists.
             *  - field annotated with @Save or @SaveWithTemplate.
             *  - setter parameter count is 1.
             */
            @Override
            public boolean matches(Method method) {
                String methodName = method.getName();
                if (!methodName.startsWith("set")) return false;
                if (method.getParameterCount() != 1) return false;

                // get target field name from method name
                String targetFieldName = Character.toLowerCase(methodName.charAt(3)) + methodName.substring(4);

                try {
                    Field targetField = method.getDeclaringClass().getDeclaredField(targetFieldName);
                    for (Class<? extends Annotation> annotationClass : annotationClasses){
                        if (targetField.isAnnotationPresent(annotationClass)) return true;
                    }
                } catch (NoSuchFieldException ignored) {}

                return false;
            }
        };
    }
}
```

여기에도 리플렉션이 덕지덕지 붙어있는데, `@Bean`이 붙은 클래스 내부에, `@Save` 혹은 `@SaveWithTemplate`가 붙은 필드들의 `setter`가 존재하고, setter 컨벤션과 시그니쳐가 일치하면 위에서 `invoke`로 정의한 AOP를 적용한다.

저장 형식은 다음과 같다.

- 템플릿종속적(@SaveWithTemplate)인 데이터 : `template-1.클래스명.필드명="값"`
- 그렇지 않은(@Save)데이터 : `클래스명.필드명="값"`

자동화한 코드가 잘 작동하는 모습을 볼 수 있었다.

![intercepter](intercept.png)
_intercept된 setter 메서드들_

![properties](property.png)
_저장된 프로퍼티 파일_

이제 Model의 필드설계가 변경되거나, 저장여부 설정은 Model에서 어노테이션 하나 추가하는것으로 끝난다.

```java
...
// 아래 필드를 영구저장하고 싶으면
private String value;

// ↓↓↓

// 어노테이션만 붙이면 더이상 할 게 없다.
@Save("초기값") private String value;
...
```

다시한번 느꼈지만, 코드변경이 편한것은 정말 압도적인 강점이었다.
