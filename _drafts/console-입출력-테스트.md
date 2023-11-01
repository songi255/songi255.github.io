---
layout: post
title: Console 입출력 테스트
description:
img_path: "/assets/img"
image:
  lqip:
  path:
  alt:
pin: false
categories: [Java, Test]
tags:
---

기본아이디어는 콘솔

JUnit에서는 아래와 같은 유틸리티 함수를 고려해볼 수 있습니다.

```java
package utils;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.PrintStream;

public class ConsoleInterceptor {
    static PrintStream standardOutputStream = System.out;
    static InputStream standardInputStream = System.in;
    static OutputStream testOutputStream;

    public static void setupConsoleIntercept() {
        testOutputStream = new ByteArrayOutputStream();
        System.setOut(new PrintStream(testOutputStream));
    }

    public static void restoreConsoleIntercept() {
        System.setOut(standardOutputStream);
        System.setIn(standardInputStream);
    }

    public static InputStream getInputStream(String input) {
        final byte[] buf = input.getBytes();
        return new ByteArrayInputStream(buf);
    }

    public static String getPrintedString() {
        String input = testOutputStream.toString();
        return input;
    }

    public static void printToConsole(String input) {
        standardOutputStream.println(input);
    }
}

```

Extension화

```java
// extension 구현

```
