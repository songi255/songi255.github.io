---
layout: post
title: "CNT 프로젝트 회고: RAW"
description:
img_path: "/assets/img"
image:
  lqip:
  path:
  alt:
pin: false
categories:
tags:
---

CNT 프로젝트 회고.

새탭이 조금 그랬고
할일 등이 분산되어있으니 집중이 깨졌다.
나는 하루 일과를 chrome으로 시작을 하는데, 여기서 모든 걸 일률적으로 관리하고 싶었다.

그래서 이런 앱을 만들었다.
크롬 스토어에 ~~가 있는데, 대충 이런 프로젝트다.(사진첨부)

특징은 custom dash board구조 + 확장 plugin 아키텍쳐 + sync 정도이다^^

프로젝트를 진행함에 있어서 설계상 맞딱들인 문제가 어떤 것들이 있었는지.. 시리즈로 공유하려고 한다.

사용 기술
CRA - typescript template
sass
@types/chrome

Agile. 확장성

0. framework 종속적인 설계구조... 디자인패턴 적용의 다양한 실패...

- data modeling...

1. storage 제약

- dao? useStorage!

use storage? proxy!

2. 동기화 문제

- sync storage? synchronizer!

3. plugin system

4. dependency? 글쎼.

- 앱 볼륨이 그렇게 크지 않은데. 굳이 크게 하는건 오버된 설계...
- chrome extension 자체가 큰 프로그램이 아니다... 가이드라인...
