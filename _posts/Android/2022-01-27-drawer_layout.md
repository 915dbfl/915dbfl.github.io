---
title: "[Android] DrawerLayout"
excerpt: "Navigation Drawer Activity 사용법!!"
categories:
  - Android
tag:
  - android 
  - kotlin
  - DrawerLayout

last_modified_at: 2022-01-27
toc: true
toc_sticky: true
search: true
---

## Intro

🙋‍♀️ 이번 포스팅에서는 안드로이드에서 기본적으로 제공하는 **Navigation Drawer Activity**사용법에 대해서 정리해보고자 한다.

해당 방식을 알기 전, 필자는 drawer 생성에 필요한 header layout과 menu 등을 직접 구현하며 복잡한 방법으로 drawer를 제작하였다.

그런데 이렇게 쉬운 방법이 있었다니...🥺
지금부터 그 사용법을 상세히 정리해보고자 한다!!

<br>

## Navigation Drawer Activity🙋‍♀️

👩 우선 새로운 액티비티를 생성할 때, Navigation Drawer Activity를 선택한다.

<img src= "https://ifh.cc/g/dsho8g.png" width = 500 height= 700> 


## 구성🙋‍♀️

👩 이제부터 자동적으로 생성된 파일들의 쓰임새를 확인해보자!

* **activity_main.xml**
  * 해당 파일을 확인해보면 아래 구성을 확인할 수 있다.

  <img src = "https://ifh.cc/g/Q8mA1B.png" width = 500 height = 600>

  * **include**: 다른 layout을 포함시키는 요소로 **프래그먼트가 교체**되는 화면을 나타낸다.
    * 최종적으로 content_main.xml의 fragment가 controller 역할을 하게 된다.

    ### - controller?👩

      * contorller는 fragment들을 관리하고, 각 fragment들의 교체를 처리한다.

  <br>

  * **navigationview**: 좌측에 나타나는 메뉴에 해당된다.
    * **headerLayout**: 좌측 메뉴 윗 부분을 구성하는 layout을 지정한다.
    * **menu**: 좌측 메뉴를 구성할 menu 파일을 지정한다.

<br>

----

* **navigation -> mobile_navigation.xml**

  * controller가 관리할 **fragment들 등록**하는 xml

  * 메뉴 아이템 클릭시, **아이템의 이름과 동일한 fragment**가 화면에 나타나게 된다. (핵심!!🔔)


  즉, 아래의 action_main_drawer.xml 속 아이템의 이름과 동일한 이름을 가진 mobile_navigation.xml의 fragment가 화면에 출력되게 되는 것이다.

  <img src = "https://ifh.cc/g/105jxS.png" width = 500 height = 600>

  <img src = "https://ifh.cc/g/VanGPp.png" width = 500 height = 600>

  
## 핵심 정리🔔

👩 결론적으로 drawer을 사용하기위해 꼭 필요한 부분만 정리하자면 다음과 같다.

* drawer에 **새로운 메뉴**를 추가하고자 한다면 menu의 <u>action_main_drawer.xml</u>에 **item을 추가**해준다.

* **동일한 이름을 가진 fragment**를 <u>mobile_navigation.xml</u>에 추가해준다.

* drawer의 **헤더 모양**을 바꾸고 싶다면 <u>activity_main.xml</u> 속  NavigationView의 **hearderLayout**을 원하는 layout을 구성해 지정한다.

<br>

## 📃참고
* 인프런 윤재성의 Kotlin 기반 안드로이드 앱 개발 Part3 수강