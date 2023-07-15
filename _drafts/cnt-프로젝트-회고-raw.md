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
추상화 : 큰 의미가 없다.

fileSystem : menifest 3 이 되면서 deprecated 되었다.

localStorage vs Indexed Storage
local이 사용하기 더 편함
sync를 위해서 어차피 stringify는 한다.
indexed 는 속도가 느리다.
용량제한 어차피 없다.
XSS 뚫리면 접근가능한거 어차피 똑같다.
chrome extension은 크롬에 종속적
보안? 몇가지 공격 포인트들이 있는데, 그건 사용자가 조심해야 함...
그리고 XSS는 대부분 chrome에서 막을 수 있다.

어차피 sync storage 대신 다른 sync 대안을 생각해봐야한다면, 큰 의미 없다.
onChanged 지원한다! sync logic도 편하다. proxy 쓸 필요도 없다.

결론 : 그냥 localStorage를 종속적으로 쌩으로 쓰는게 맞다고 판단했다.

2. 동기화 문제

- sync storage? synchronizer!

3. plugin system

fileSystem 이 안된다.
남은 대안은 localStorage에 fetch하여 저장 후, 파일로 바꿔서 읽어들이는것...

4. dependency? 글쎼.

- 앱 볼륨이 그렇게 크지 않은데. 굳이 크게 하는건 오버된 설계...
- chrome extension 자체가 큰 프로그램이 아니다... 가이드라인...
