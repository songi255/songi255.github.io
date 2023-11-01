---
layout: post
title: Java 함수 실행시간 측정의 함정
description:
img_path: "/assets/img"
image:
  lqip:
  path: stopwatch.jpg
  alt: Pixabay로부터 입수된 Gino Crescoli님의 이미지 입니다.
pin: false
categories:
tags:
---

**100번 루프와 10만번 루프 중 어느것이 더 빠를까요?** 바보가 아니라면 쉽게 알 수 있습니다.

테스트 해보겠습니다.

```java
@RepeatedTest(1000)
void test1() {
    long start = System.nanoTime();
    // loop 100 times
    for (int i = 0; i < 100; i++) {
        Object obj = new Object();
    }
    long end = System.nanoTime();

    // collect execution time
    test1List.add((end - start));
}

@RepeatedTest(1000)
void test2() {
    long start = System.nanoTime();
    // loop 100,000 times
    for (int i = 0; i < 100000; i++) {
        Object obj = new Object();
    }
    long end = System.nanoTime();

// collect execution time
    test2List.add((end - start));
}
```

1000회 반복 테스트 결과는 다음과 같습니다.

(사진)

테스트2가 더 빠르네요. 이를 통해 루프를 10만번 돌리는 것이 100번 돌리는 것 보다 훨씬 빠르다는 것, 그리고 제가 바보라는 것을 알게 되었습니다. 앞으로 루프는 꼭 10만번씩 돌려야겠군요.

**물론 절대로 그렇지 않습니다.** 제가 무언가를 놓치고 있나요? 실행시간 측정에는 많은 함정들이 숨어있습니다. 지금과 같은 작은단위의 벤치마킹에서 훨씬 위험하며, 특히 학습중이었다면 자칫 잘못된 지식을 가지게 될 수 있습니다.

이런 방식의 성능테스트는 과연 유의미할까요? 해당 주제에 대한 몇가지 정보와, 저의 생각을 정리해보았습니다.

## 1. JVM의 함정

### 1-1. 최적화로 인한 오류

Java 코드는 다음 과정을 거쳐서 실행됩니다.

1. 컴파일 : 소스코드 `~.java`를 바이트코드 `~.class`로 변환합니다.
2. 실행 : 변환된 바이트코드는 JVM에서 `로딩 -> 검증 -> 기계어 번역`을 통해 최종적으로 타깃 OS에서 실행됩니다.

문제는 **기계어 번역과정**에서 발생합니다.

기계어 번역은 **JIT(Just-In-Time) Compiler**가 담당하는데, 이때 상당한 최적화가 발생합니다. 예를들면

- 함수가 인라인 삽입되어 호출스택이 줄어듭니다.
- 인라이닝을 통해 for 루프를 벗기거나 루프 반복횟수를 줄입니다.
- 사용되지 않는 변수를 제거합니다.
- 자주사용되는 코드는 네이티브 메서드로 컴파일되어 JVM 코드영역에 캐싱됩니다.

이런 정보들을 통해 테스트결과의 원인을 추론해볼 수 있습니다.

- for 루프는 벗겨졌고, 여러개의 `Object obj = new Object();`는 인라이닝 됩니다.
- obj는 사용되지 않기 때문에 최적화됩니다.
-

ㄷㅈㄻ실제로 확인하고 싶다면 JVM 로그를 켜서 확인하세요.

### 1-2. GC에 의한 오류

가비지 수집

## 2. 캐시의 함정

클린코드가 느린 이유. CPU

## 3. 실행환경의 함정

하드웨어 환경 및 상태(온도 등), 프로세스 환경

## 4. 해결책

### 4-1. JMS

### 4-2. 프로파일링

## Reference

- https://web.archive.org/web/20200606234821/https://www.ibm.com/developerworks/java/library/j-jtp02225/
- https://stackoverflow.com/questions/504103/how-do-i-write-a-correct-micro-benchmark-in-java
- https://www.oracle.com/technical-resources/articles/java/architect-benchmarking.html
- https://www.ibm.com/docs/en/sdk-java-technology/8?topic=compiler-how-jit-optimizes-code
- https://www.geeksforgeeks.org/compilation-execution-java-program/
