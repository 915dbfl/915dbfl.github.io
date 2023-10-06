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

- 시스템에서 유지하는 토큰 참조
- 애플리케이션 b에서 a가 살아있는지와는 상관없이 a를 대신에 사전에 정의된 작업을 실행할 수 있도록 pendingIntent를 전달할 수 있다.

=> 그렇다! <span style = "background-color:#fff5b1">AlarmManager가 백그라운드 작업</span>이기 때문에 백그라운드 작업에게 pendingIntent를 통해 작업을 넘겨주게 되는 것이다.

<br>

## 👩🏻‍💻 PendingIntent의 사용 사례: 크게 세가지
1. 알림 - 작업 수행
    - NotificationManager가 Intent 실행
2. 앱위젯 작업 실행
    - 메인 화면 앱이 Intent 실행
3. 지정된 시간 - 작업 수행
    - AndroidManager가 Intent 실행

<br>

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
    - PendingIntent.getBroadcast

→ 각 메서드는 다음 세 개의 인자를 받는다.

- 현재 앱의 context
- 감싸고자 하는 Intent
- 인텐트의 적절한 사용방식 Flag
    - [flag 종류 알아보기](https://developer.android.com/reference/android/app/PendingIntent#summary)

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