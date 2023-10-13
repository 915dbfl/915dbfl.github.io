---
title: "[PendingIntent/LaunchMode] 알림 받고 활동 시작!"
excerpt: "알림을 눌렀을 때 활동을 시작하는 방법"
categories:
  - Android
tag:
  - android
  - PendingIntent
  - launchMode

last_modified_at: 2023-10-12
toc: true
toc_sticky: true
search: true
---

## 👩🏻‍💻 활동의 구분

AlarmManager와 Broadcast Reciever를 활용해 알림을 생성하고 받았다면, <span style = "background-color:#fff5b1">알림을 눌렀을 때 `PendingIntent`를 활용해 새로운 활동</span>을 정의할 수 있다. 활동은 크게 다음 두가지로 구분되며, 그에 따라 공식 문서 가이드를 따라하면 된다.

### 1. 일반 활동
>  `앱의 일반 ux 흐름의 일부로 존재하는 활동`, 즉, <span style = "background-color:#fff5b1">활동에 백 스택이 포함되어 있어야 한다.</span>

### 2. 특수 활동
> `알림에서 시작할 때만 해당 활동을 볼 수 있다.` 즉, <span style = "background-color:#fff5b1">백 스택을 포함하지 않으며</span>, 알림 자체에서 제공하기 어려운 추가 정보를 제공하게 된다.

<br>

## 👩🏻‍💻 알림으로 fragment를 시작할 순 없을까?

필자의 경우, 알림을 클릭했을 떄 특정 fragment로 이동했어야 했다. 그렇다면 pendingIntent를 통해 `fragment`로 이동할 순 없을까?

구글링을 해본 결과, <span style = "background-color:#fff5b1">직접 fragment로 화면 전환을 할 수 없다.</span> 따라서 fragment로 전환하고자 한다면 <span style = "background-color:#fff5b1">해당 fragment를 포함하는 activity로 전환한 후, 원하는 fragment로 전환을 처리해야 한다.</span>
* <https://stackoverflow.com/questions/71967698/how-to-open-fragment-from-the-click-of-the-notification>

activity로 전환하는 것은 pendingIntent의 `getActivity`를 활용할 수 있다.
```kotlin
            val mainIntent = Intent(this, MainActivity::class.java)
            mainIntent.putExtra("preorderId", preorderId)
            PendingIntent.getActivity(
                this,
                UUID.randomUUID().hashCode(),
                mainIntent,
                PendingIntent.FLAG_IMMUTABLE
            ).send()
```

<br>

## 👩🏻‍💻 액티비티가 두 번 생성되는 현상?

pendingIntent를 통해 getActivity를 활용할 경우, 새로운 activity가 생성된다. <span style = "background-color:#fff5b1">그렇다면 해당 activity가 실행되고 있다면 새로운 액티비티를 생성하지 않고, 기존 액티비티를 사용할 순 없을까?</span>

이때 활용할 수 있는 것이 `launchMode`이다. 다음 게시글에서 각 launchMode에 대해 자세히 다루고 있다! 필자는 그 중 `singleInstance`를 적용해 해당 문제를 해결할 수 있었다.
* <https://mashup-android.vercel.app/mashup-11th/heejin/launchMode/launchMode/>

<br>

## 👩🏻‍💻 참고
* <https://developer.android.com/training/notify-user/navigation?hl=ko>