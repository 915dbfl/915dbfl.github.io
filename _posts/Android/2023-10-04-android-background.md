---
title: "[Android] 백그라운드 작업"
excerpt: "즉시? 지연? 정시?"
categories:
  - Android
tag:
  - android
  - background

last_modified_at: 2023-10-04
toc: true
toc_sticky: true
search: true
---

해당 문서는 [안드로이드 백그라운드 처리 가이드 공식 문서](https://developer.android.com/guide/background?hl=ko)를 참고해 작성한 글입니다.

<br>

## 👩🏻‍💻 어떤 게 백그라운드 작업인데?

본격적으로 백그라운드 작업에 대해 다루기 전에 `어떤 게 백그라운드 작업인데?`라고 궁금증을 갖는 사람이 분명 있을 것이다. 

> 기본 원칙
  일반적으로, 몇 밀리초 이상 걸리는 작업은 백그라운드 스레드에 위임해야 합니다. 일반적으로 장기 실행 작업에는 비트앱 디코딩, 저장소 엑세스, 머신러닝 모델 작업, 네트워크 요청 실행 등이 포함됩니다.

<br>

## 👩🏻‍💻 백그라운드 작업 처리

백그라운드 작업은 크게 <span style = "background-color:#fff5b1">즉시, 지연, 정시</span> 분류되며 공식 홈페이지에서 각 상황에 맞게 처리되어야 하는 방식들이 제안되어 있다.

### 1. 즉시
* 바로

  * `kotlin coroutine`

* 미디어 재생

  * `foreground 처리 권장`

* 백그라운드 전화하거나 기기 재시작 이후 즉시 실행해야 하는 경우

  * `workManager`

### 2. 지연
* `workManager`

### 3. 정시
* `AlarmManager`