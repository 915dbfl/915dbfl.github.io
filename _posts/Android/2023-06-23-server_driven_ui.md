---
title: "[SWM] Server Driven UI - 개념 정리"
excerpt: "서비스 변화에 유연한 백엔드-모바일 구조 설계"
categories:
  - Android
tag:
  - android
  - server driven ui
  - SWM
  - Mobile Camp

last_modified_at: 2023-06-28
toc: true
toc_sticky: true
search: true
---

# 👩🏻‍💻 모바일 서비스 업데이트 시 문제점

`모바일 서비스 업데이트 -> 앱 배포`

이때 문제점은 바로 유저들은 쉽게 업데이트를 받아주지 않는다는 점이다!
그렇다면 
<span style="background-color:#fff5b1">**앱 배포를 거치지 않고 서비스에 변화를 줄 수는 없을까??**</span>

그게 바로  `Server Driven UI`이다. 즉, 서버에서 화면을 컨트롤할 수 있게 되는 것이다.

<br>
  
# 🤨 강제 업데이트 하면 되지 않나요?
특정 버전 이하일 경우, 강제 업데이트를 하도록 처리한다 가정하자! 이렇게 될 경우, <span style="background-color:#fff5b1">**사용자는 특정 목적으로 우리의 앱에 들어왔을 것이다.**</span> 하지만 업데이트 화면으로 넘어가버리기 때문에 ux를 해치는 것이다!

<br>

# 🤔 프론트엔드에서 화면을 컨드롤 하지 않는 방법: Feature Flag & A/B Test

-> `Feature Flag?`
  * 새롭게 구현했거나, 개선시킨 기능을 특정 경우에만 열 수 있게 해주는 기능
    * 배포/출시 시점 ≠ 노출 시점
    *  간단히 `firebase remote config`를 통해 적용할 수 있다. 직접 활용 해본적은 없지만 특정 변수 값을 저장하고 firebase에 저장된 값이 변경됨에 따라 로직을 처리할 수 있게 된다.

-> `A/B Test`
  * Feature Flag를 응용하여 비즈니스의 개선 방향을 결정한다.
    * 똑같은 기능이지만 배치, 모양 등이 서로 다른 방안을 준비한다.
    * 이 또한 `firebase remote config`를 통해 적용할 수 있다. feature config로 만들어진 여러 대안을 %를 정해 특정 사용자에게만 테스트를 할 수 있게 된다.

## Feature Flag & A/B Test의 장점
* 배포/출시 ≠ 노출
* 적은 노출/리스크 -> 개선 방향 결정
* 리스크가 감지된다면 빠른 롤백이 가능

## Feature Flag & A/B Test의 단점
* a/b 코드 분기로 인한 관리가 어렵고, 결정에 따라 지속적으로 a/b 코드의 clean up이 필요하다.
* 여러 개의 a/b가 서로 연관이 되었다면 영향도 파악이 어렵다.

<br>

# 🤨 Feature Flag, A/B Test - Intro API

`Intro API`
* 앱이 cold start되는 시점
* 서버로부터 앱 구동시 세팅되어야 하는 데이터를 내려주는 API

## 언제?
앱이 splash 화면이 있을 경우? api의 response가 돌아온 뒤 메인 화면으로 이동하도록 처리한다.

-> feature flag, a/b test, 강제 업데이트 안내 팝업

<br>

# 🤨 Server Driven UI?

위에서 Feature Flag나 A/B Test를 통해 화면 컨트롤을 클라이언트쪽에 의존하지 않는 방법에 대해 살펴봤다!

A/B Test를 할 경우, A냐 B냐에 따랴 로직을 분리해야 하므로 서버의 response는 다음과 같을 것이다!
```
{
  "responseData": {
      "screenName": "Intro",
      ...

      {
        "title": "...",
        "value": "A"
      },
      {
        "title": "...",
        "value": "B"
      }
  }
}
```

그럼 우리는 나아가 이런 생각까지 할 수 있다! <span style="background-color:#fff5b1">'어? 서버쪽에서 A냐 B냐를 알려주는 것에서 나아가 어떤 뷰를 그려야 하는지를 알려줄 순 없을까?'</span> 그 개념이 바로 `Server Driven UI`이다!!

시작할 때 잠깐 언급했듯이 Server Driven UI는 서버에서 화면을 컨트롤할 수 있도록 하는 방식이다. 
* 장점: 백엔드 배포만으로 콘텐츠 변경을 제공할 수 있다.
* 단점: 백엔드에서도 디자인 시스템을 구성하고, 그 변화를 지속적으로 관리해야 한다.


<br>

# 🤨 Server Driven UI 고려사항

## 1. 서버에 제공해야 하는 정보들
백엔드에서 제공하는 리소스로부터 빠르고 잦은 기능 변화를 유저들에게 제공하기 위해서는 **모바일 상태를 파악하기 위한 정보**를 **network request header**로 전달해줘야 한다.

대표적인 것들 몇 개를 정리하면 다음과 같다.
* App-Version:
  해당 앱의 버전을 전달
* Device-OS:
  해당 디바이스의 모델명, OS, 버전 등
* Language
  Locale에 매치되는 language, 글로벌향을 지향할 경우, 해당 정보가 꼭 필요하게 된다.

## 2. 에러 처리
**서버에 의해 화면이 컨트롤된다고 할 때, 당연히 그에 대한 에러 처리가 중요해진다.** 서버에서 에러가 발생했다고 ux가 끊기게 된다면 최악의 상황이다.

따라서 우리는 서버의 에러를 단순히 에러를 던져주는 것으로 처리하지 않고, <span style="background-color:#fff5b1">**에러가 났을 때 null이나 unknown을 표시하는 방향**</span>으로 유저의 ux가 끊기지 않도록 해야 한다.

<br>

# 😌 정리

그렇다면 왜 앱 배포만으로 업데이트 하는 것이 문제가 되는지? 이를 위해서는 어떤 방식을 사용할 수 있는지?를 개념적으로 쭉 살펴보았다!

그렇다면 이제 본격적으로 Server Driven UI 작업을 진행해보자! 그리고 그 과정에서는 어떠한 부분은 신경써야 하는지 파악해보자!

<br>

## 📃참고
* SWM 모바일 특강