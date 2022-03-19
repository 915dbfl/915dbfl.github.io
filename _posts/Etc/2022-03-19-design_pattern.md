---
title: "[디자인 패턴] MVC/MVP/MVVM"
categories:
  - Etc
tag:
 - DesignPattern
 - MVC
 - MVP
 - MVVM

last_modifeid_at: 2022-03-19
toc: true
toc_sticky: true
search: true
---

<img src = "https://ifh.cc/g/JXFcrq.png" width = 600 height = 500>

## 👩디자인 패턴

디자인 패턴의 주된 목적은 **'역할을 나눠서 코드를 관리'**하는 것이다.

데이터 선언, 데이터 처리 등 모든 것이 한 곳에서 이루어지게 되면 어플리케이션이 복잡해질수록 그 과정도 복잡해지게 된다.

따라서 이를 해결하기 위해서 적용되는 개념이 디자인 패턴이다.

오늘은 디자인 패턴 중에서 **MVC, MVP, MVVM** 세가지에 대해서 알아보고자 한다.

<br>

## 🙋‍♀️ Model, View

세가지 디자인 패턴에 공통적으로 들어가는 Model, View가 무엇인지에 대해서 살펴보자.

* **Model**: 테이터 상태, 비즈니스 로직 부분으로 테이터를 보관하고 필요에 따라 가공하기도 한다.

* **View**: 사용자들에게 보여지는 UI 부분이다. UI 요소에 대한 이벤트(버튼을 누르거나, 글자를 입력하는 등의 행위)가 발생한다.

<br>

👩 그렇다면 지금부터 각 패턴을 하나씩 살펴고자 한다.

<BR>

## ☝️MVC: Model + View + Controller

<img src = "https://ifh.cc/g/Mv98ay.jpg" width = 600 height = 500>

* controller: **Action(사용자의 입력, 화면 이동 등)이 발생했을 때** 이를 받고 처리를 담당한다.

### 🙋‍♀️동작과정
  * Action이 Controller에 들어오게 된다.
  * Controller는 Action을 확인하고 Model을 업데이트한다.
  * Controller는 Model을 나타낼 View를 선택한다.
  * Model을 통해 View가 화면에 나타난다.

### 🙋‍♀️특징
  * Controller : View = 1 : N
  * Controller가 업데이트를 직접 하지는 않는다.

### 🙋‍♀️장단점
* 장점: 가장 널리 사용되는 패턴으로 가장 단순하다.
* 단점: <u>View와 Model 사이 의존성이 높아</u> 어플리케이션이 복잡해질경우 유지보수가 어렵다.

<br>

## ✌️MVP: Model + View + Presenter

<img src = "https://ifh.cc/g/PWSyGN.jpg" width = 600 height = 500>

* Presenter: Controller와 기능이 유사하나 View에서 요청한 정보로 Model을 가공하여 View에게 전달해준다. View와 Mode 사이를 이어주는 다리?로 생각하자!

### 🙋‍♀️동작과정
  * Action이 View로 들어온다.
  * View에서 Presenter에게 데이터를 요청하고 Presenter는 Model에게 데이터를 요청한다.
  * Model에서 Presenter의 요청에 응답하고, Presenter가 View의 요청에 응답한다.
  * View에서 Presenter를 통해 얻은 데이터를 화면에 나타낸다.

### 🙋‍♀️특징
  * Presenter : View = 1 : 1

### 🙋‍♀️장단점
* 장점: View와 Model의 의존성이 없다. Presenter를 사이에 두고 데이터를 주고받기 때문이다.
* 단점: **View와 Presenter 사이의 의존성**이 강하다.

<br>

## 👌MVVM: Model + View + ViewModel

<img src = "https://ifh.cc/g/37dlO5.jpg" width = 600 height = 500>

* ViewModel: 단어 그대로 View를 위한 Model이다. View를 나타내기 위한 데이터를 처리하게 된다.

### 🙋‍♀️동작과정
  * Action이 View로 들어온다.
  * View에서 ViewModel로 Action을 전달한다.
    * 데이터 바인딩이 되면 ViewModel이 감지하게 된다.
  * ViewModel이 Model에게 데이터 변경 처리를 요청한다.
  * Model이 ViewModel에게 요청받은 데이터를 응답한다.
  * ViewModel은 받은 데이터를 가공해 저장한 후 결과를 알린다.

### 🙋‍♀️특징
  * **Command 패턴**과 **데이터 바인딩**을 통해서 View와 ViewModel 사이의 의존성을 없앴다.

### 🙋‍♀️장단점
* 장점: 각 부분이 **독립적**이기 때문에 모듈화하여 개발할 수 있다.
* 단점: 설계가 쉽지 않다.

<br>


## 📃 참고
* [https://beomy.tistory.com/43](https://beomy.tistory.com/43)
* [https://jason-api.tistory.com/36](https://jason-api.tistory.com/36)