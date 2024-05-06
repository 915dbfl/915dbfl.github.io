---
title: "[PendintIntent] 다른 앱에게 intent 전달하기"
categories:
  - Android
tag:
  - android
  - AlarmManager

last_modified_at: 2023-10-06
toc: true
toc_sticky: true
search: true
---

AlarmManager를 사용하다보면 pendingIntent를 만들어 AlarmManager에게 전달해주는 부분이 있다. 

'Intent는 아는데, pendingIntent는 뭐야??🤔' 

<br>

## 👩🏻‍💻 PendingIntent 개념 핵심

- 사전적 의미: <span style = "background-color:#fff5b1">당장 수행하지는 않고 특정 시점에 수행하는 Intent</span>
- 역할
    - pendingIntent를 받은 대상은 <span style = "background-color:#fff5b1">마치 자신의 intent인 것처럼 작업을 수행할 권한을 얻는다.</span>

=> 그렇다! 앱에서 AlarmManager를 통해 notification을 생성한다면 <span style = "background-color:#fff5b1">이는 앱이 아닌 다른 대상에게 intent를 전달하는 것이기 때문에</span> pendingIntent를 활용하는 것이다.
<br>

## 👩🏻‍💻 PendingIntent의 사용 사례: 크게 세가지
1. 알림 - 작업 수행
    - NotificationManager가 Intent 실행
2. 앱위젯 작업 실행
    - 메인 화면 앱이 Intent 실행
3. 지정된 시간 - 작업 수행
    - AlarmManager가 Intent 실행

<br>

위의 사용 사례를 통해 알 수 있듯이, 앱과 상관없이 특정 시점에 다른 누군가가 intent를 수행해주게 할 때 pendingIntent를 활용한다.

## 👩🏻‍💻 유형에 따른 PendingIntent 생성
### 유형에 따른 PendingIntent 생성

1. intent 객체는 앱 구성 요소(activity, service, broadcastReceiver)가 처리하도록 설계되어 있다.
2. 따라서 앱에서 pendingIntent를 실행할 경우, startActivity와 같이 call이 있는 intent를 실행시키지 않는다.

→ pendingIntent 생성 유형

- activity를 시작하는 intent의 경우
    - PendingIntent.getActivity()
- service를 시작하는 intent의 경우
    - PendingIntent.getService()
- broadcastReceiver를 시작하는 Intent의 경우
    - PendingIntent.getBroadcast()

→ 각 메서드는 다음 세 개의 인자를 받는다.

- 현재 앱의 context
- 감싸고자 하는 Intent
- 인텐트의 적절한 사용방식 [Flag](https://developer.android.com/reference/android/app/PendingIntent#summary)
    - FLAG_CANCEL_CURRENT
        - 이전에 생성한 pendingIntent 취소 후 새로 생성
    - FLAG_NO_CREATE
        - 이전에 생성한 pendingIntent 있을 시, 재활용 (없으면 null)
    - FLAG_ONE_SHOT
        - 일회성으로 pendingIntent 사용
    - FLAT_UPDATE_CURRENT
        - 기존 pendingIntent 업데이트해 사용

<br>

## 👩🏻‍💻 Flag 설정 시, 주의할 점

Flag를 사용하다 해당 오류가 발생했다.

```
Targeting S+ (version 31 and above) requires that one of FLAG_IMMUTABLE or FLAG_MUTABLE be specified when creating a PendingIntent.
```

즉, <span style = "background-color:#fff5b1">타겟팅 version이 31 이상이라면 FLAG_IMMUTABLE, FLAG_MUTABLE 중 하나를 사용해야 한다.</span>

<br>

## 👩🏻‍💻 참고
- <https://developer.android.com/reference/android/app/PendingIntent>
- <https://velog.io/@thevlakk/Android-AlarmManager-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-1>