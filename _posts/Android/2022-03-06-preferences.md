---
title: "[Android] Preferences"
excerpt: "application 설정 data 저장하는 방법!"
categories:
  - Android
tag:
  - android 
  - kotlin
  - preferences

last_modified_at: 2022-03-06
toc: true
toc_sticky: true
search: true
---

## 👩Preferences
* 안드로이드의 저장 방식 중 하나
  * 다수의 매개체에 대한 data와 같은 **많은** 양의 데이터 저장 : **SQLite**
  * application 설정 data와 같은 유일하고 **적은** 데이터 저장 : **preferences**

<br>

## 👩Preferences Screen
* **사용자**에게 **환경 설정 화면**을 제공할 경우 사용할 수 있다.
* **UI를 통해서** preferences를 사용할 수 있도록 제공되는 개념이다.
* **preferenceFragment**를 사용하며 저장기능까지 모두 구현되어 있다.

<br>

## 🙋‍♀️Preferences에 데이터 저장 및 불러오기

* 제공되는 preferences screen을 사용하기 이전에 특정 데이터를 preferences에 저장하고 불러오는 과정을 먼저 살펴보자.

### ☝ 데이터 저장하기
<img src = "https://ifh.cc/g/4E75xL.png" width = 600 height = 700>

* preferences 객체 추출 시 모드
  * **MODE_PRIVATE**: 모든 정보 삭제 후 재저장
  * **MODE_APPEND**: 덮어 씌우기

---

### ✌ 데이터 불러오기
<img src = "https://ifh.cc/g/qY6py4.png" width = 600 height = 700>

<br>

## 🙋‍♀️Preferences Screen 사용하기

### ☝ 1. preferences library 설정하기

* 상단의 File - project structure 클릭!
* preferences library dependency 추가하기
  <img src = "https://ifh.cc/g/9ZGBlz.png" width = 600 height = 500>
  * 해당 사진까지 진행한 후, library dependency를 눌러 preference를 검색해 해당 library를 적용한다.

<br>

* 위 과정이 오래걸린다면 직접 library를 설정할 수 있다. (module: gradle 파일)
<img src = "https://ifh.cc/g/ycZ7gw.png" width = 600 height = 500>

---

### ✌ 2. xml directory 생성하기
* res directory - **xml directory** 생성하기
* xml resource file 추가하여 **preferenceScreen** 만들기

---

### 👌 3. preferences screen 설정하기
* 사용자가 환경 설정을 하기 위해 필요한 버튼, 스위치 등을 가지고 직접 xml을 구성할 필요가 없다.
* <u>두 번째 단계</u>에서 생성한 xml resource file이 사용자가 환경 설정을 하는데 사용될 xml이 된다.

  **🌟여기가 중요🌟**

  <img src = "https://ifh.cc/g/qc8UCf.png" width = 600 height = 400>
  * 해당 xml이 보여질 activity는 **PreferenceFragmentCompat()**을 상속해야 한다.

---

### ✋ 4. preference screen에 다양한 요소 추가하기

요소 중 두 개의 요소의 설정값들을 살펴보며 각 설정이 어떠한 역할을 하는 지 살펴보겠다!

<img src = "https://ifh.cc/g/zXwA8S.png" width = 400 height = 400>

<img src = "https://ifh.cc/g/bRHEF7.png" width = 400 height = 400>

|  설정  |  값  |
|:--------:|:------:|
|**default value**| **최초로 나타났을 때** 자동으로 저장되는 값으로 변경할 경우 재설치를 해야 한다.|
|**key**| data **저장 시** 사용할 이름으로 **불러올 때**도 해당 값을 사용한다.|
|**title**| 환경 설정 화면에 해당 요소가 **표시**될 이름|
|**summary**| title 밑에 나타날 요약 문자열|
|**summaryOn**| summary는 무시되고, **on**일 경우 나타나는 요약 문자열(스위치)|
|**summaryOff**| summary는 무시되고, **off**일 경우 나타나는 요약 문자열(스위치)|
|**singleline**| true일 경우, 한 줄만 입력이 가능하다.|
|**dependency**| **설정한 요소**가 true/false냐에 따라 해당 요소가 활성/비활성이 된다. (dependency로 주로 **switch**를 많이 사용한다.)|

 

<br>

---

👩 여기서 잠깐❕❕ 리스트 요소의 경우 알아야 할 부분이 있어 따로 살펴보자❕❕

### 🔔 리스트 요소: 단일/ 다중 선택

* 리스트 요소를 추가할 경우, 단일/ 다중 선택이 가능한 리스트를 골라 추가할 수 있다.

  **여기서 중요한 점!**

* 리스트 요소를 사용할 때 팝업으로 창이 뜨면서 해당 값들을 설정해야 한다.
  * **entries resource**: 리스트 선택 시 화면 상에 보여질 문자열들
  * **entry values resource**: 화면 상에 보여지는 요소들에 매칭되는 실제 코드에서 사용될 값들
  * **defaultvalue resource**(다중 선택 사용 시): 다중 선택에서 처음에 선택되어질 값들

* 해당 값들은 **strings.xml**에 다음과 같이 추가한다.
<img src = "https://ifh.cc/g/QLafei.png" width = 500 height = 600>

---

### 🖐 5. preferences에서 저장된 값 가져오기

👩 이렇게 설정한 요소들은 값이 변경될 때마다 자동으로 preferences에 저장이 된다. 그렇다면 마지막으로 이렇게 저장된 값들을 가져오는 과정을 살펴보자!

<img src = "https://ifh.cc/g/lqJduu.png" width = 600 height = 700>

~~생각보다 내용이 길어져서 두개의 포스팅으로 나눌까 고민했지만 내용만 본다면 어려울게 없는 내용이라 한 포스팅으로 마무리한다!! 오늘도 끝!!~~

## 📃참고
* 인프런 윤재성의 Kotlin 기반 안드로이드 앱 개발 Part3 수강