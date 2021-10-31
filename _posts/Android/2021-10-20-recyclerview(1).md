---
title: "[Android] RecyclerView 사용하기(1)"
excerpt: "RecyclerView이란? / RecyclerView 구현 플로우"
categories:
  - Android
tag:
  - android 
  - RecyclerView
  - android recyclerview
  - kotlin

last_modified_at: 2021-10-31
toc: true
toc_sticky: true
search: true
---

## Intro

이번 시간에는 **recyclerview**를 사용해 스크롤이 가능한 목록을 구현하고자 한다. 구현에 앞서 recyclerview가 무엇이고, 어디에 적용가능한지 등을 살펴보자!

[[RecyclerView로 동적 목록 만들기]](https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ko)

필자는 recyclerview에 대한 안드로이드 공식 문서를 참고해 해당 게시글을 작성한다!

<br>

## 🙋‍♀️ RecyclerView

* 우리는 recyclerview를 통해서 **대량의 데이터 세트를 효율적으로 표시**할 수 있다. 

* 개발자가 recyclerview에 목록으로 표시될 **데이터를 제공**하고, **각 항목의 모양을 정의**하면 recyclerview 라이브러리에서 필요한 요소를 동적으로 생성하게 된다.

<br>

### 👩 여기서! recyclerview의 특징?

* 이름을 통해서 알 수 있듯이 **요소들을 재활용**한다는 점이다!

  * 스크롤에 의해 화면에서 사라진 뷰를 제거하지 않고, 그 뷰를 재사용하여 새항목의 뷰를 구성한다.

  * 따라서 앱의 응답성은 좋아지고, 전력 소모는 줄어들게 된다.

* 수직만을 제공하는 리스트뷰와는 달리 **수직, 수평 방향 모두 제공**한다.

<br>

---
### 🙋‍♀️ 구성 요소

우선 recyclerview를 구현하기에 앞서 각 구성 요소가 어떤 역할을 하는지에 대해 살펴보고자 한다.

* ["https://recipes4dev.tistory.com/154"](https://recipes4dev.tistory.com/154)
  * 해당 게시글을에 각 요소들의 역할, 구현 과정에 대해 정말 자세히 정리되어 있다. recyclerview를 학습하면서 해당 게시글을 참고하길 바란다.


* **어댑터**: 주어진 데이터 목록을 하나의 아이템 단위의 뷰로 생성하는 역할을 한다.

* **레이아웃 매니저**: recyclerview는 수직, 수평 방향뿐만 아니라 **다양한 형태의 레이아웃**을 직접 커스터마이징할 수 있다. 이때 레이아웃 매니저는 아이템이 화면에 표시될 때, 어떻게 **배치**를 할 것인지 그 형태를 관리하게 된다.

* **viewholder**: 화면에 표시될 아이템 뷰를 저장하는 객체이다. 
  * 목록의 데이터 요소는 뷰 홀더 객체로 정의된다. 
  * 뷰 홀더가 생성되었을 때는 뷰 홀더에 연결된 데이터가 없다. 뷰 홀더가 생성된 후, recyclerview가 뷰 홀더를 뷰 데이터에 바인딩한다.

<br>

---
### 🙋‍♀️ recyclerview 플로우
![플로우](https://ifh.cc/g/nBKJQS.jpg)
해당 사진이 나에게는 recyclerview가 어떻게 진행되는지 이해하는데 많은 도움이 되었다.

사진을 바탕으로 플로우를 한 번 정리해보겠다!!
* 사용자가 제공한 데이터 목록의 각 데이터는 어댑터를 통해 하나의 뷰로 구성된다.

* 이때 layoutmanager에 지정된 레이아웃 형태가 뷰에 적용이 된다.

* 만들어진 각 데이터의 뷰는 뷰 홀더를 통해서 recyclerview에 위치하게 된다.


생각보다 간단한 방식이지 않은가??🙃

<br>

## 👩 Recyclerview 구현 과정

recyclerview를 사용하기 위해서 우리가 진행해야 하는 과정은 다음과 같다.



* 화면에 **recyclerview 요소 추가**하기

* recyclerview를 구성할 **item 레이아웃 추가**하기

* recyclerview에 적용할 **adapter** 구성하기

* 화면 속 **recyclerview 요소**에 <u>어댑터와 레이아웃매니저</u> 지정하기

<br>

그렇다면 본격적인 구현은 [다음 게시글](http://127.0.0.1:4000/android/recyclerview(2)/)에서 진행하자!!

이번 게시글은 이만 

끝~ ~(˘▾˘~)

<br>

## ✍ GIT REPO
* [https://github.com/915dbfl/youlAndroid](https://github.com/915dbfl/youlAndroid)

<br>

## 📃참고
* https://recipes4dev.tistory.com/154
* https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ko
