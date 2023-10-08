---
title: "[AlarmManager] 알람 매니저 사용법"
excerpt: "알람 매니저로 알림 등록 -> 브로드캐스트로 후처리!"
categories:
  - Android
tag:
  - android
  - AlarmManager

last_modified_at: 2023-10-09
toc: true
toc_sticky: true
search: true
---

## 👩🏻‍💻 AlarmManager

그렇다면 본격적으로 알람 매니저를 사용하는 방법에 대해 다뤄보고자 한다. 사용법이 비교적 매우 간단하다!

그 과정을 정리해보면 다음과 같다.
1. 알람을 받을 broadcast receiver를 정의한다.
2. 정의한 broadcast receiver를 등록한다.
3. 각 알람에 대한 pendingIntent를 정의한다.
4. alarmManager에 해당 pendingIntent를 전달한다.
5. broadcast receiver에 의해 알람을 받는다.
6. 정의된 후처리를 진행한다.

## 1. broadcast receiver 정의

```kotlin
class AlarmReceiver : BroadcastReceiver() {
    override fun onReceive(p0: Context?, p1: Intent?) {
        // 추후 처리 정의
    }
}
```

<br>

## 2. broadcast receiver 등록

broadcast receiver를 등록하는 방법은 크게 두 가지가 있다.
1. manifest에 등록하는 방법
2. activity에 등록하는 방법

각 방법의 차이점은 <span style = "background-color:#fff5b1">앱이 실행 중이냐</span>에 따라 달라진다.
1. manifest에 등록하는 방법
  - 앱이 실행 중이지 않을 때도 broadcast receiver가 동작한다.
2. [activity에 등록하는 방법](https://developer88.tistory.com/entry/Broadcast-Receiver-%EB%93%B1%EB%A1%9D%ED%95%98%EA%B3%A0-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%B0%9B%EC%95%84%EC%84%9C-%EC%B2%98%EB%A6%AC%ED%95%98%EA%B8%B0)
  - 등록한 activity의 lifecycle를 따르게 된다.
  - 단, 등록한 broadcast를 제거해줘야 메모리 낭비가 되지 않는다.

```kotlin
        // manifest를 이용한 등록의 경우
        <receiver
            android:name=".common_ui.receiver.AlarmReceiver"
            android:enabled="true" />
```
* 여기서 `enabled`를 <span style = "background-color:#fff5b1">false</span>으로 설정하면 앱 외부 소스의 브로드캐스트를 수신하지 않는다.

<br>

## 3. 알림에 대한 pendingIntent 정의 및 alarmManager에게 전달

```kotlin
    fun setPreorderAlarm() {
        // 알람 매니저 가져오기
        val alarmManager = context.getSystemService(Context.ALARM_SERVICE) as AlarmManager

        // 현재 시간 이후인 지 확인 후 알람 등록
        val currentTime = Calendar.getInstance().time.getUTCDateTime()
        val alarmTime = Formatter.getDateFromString(preorderDate)
        if (alarmTime.after(currentTime)) {
            val alarmIntent = Intent(context, AlarmReceiver::class.java)
            // intent를 활용해 원하는 값 넘김
            alarmIntent.putExtra("preorderDate", preorderDate)
            alarmIntent.putExtra("phoneNumber", phoneNumber)
            alarmIntent.putExtra("totalOrderCount", totalOrderCount)
            // broadcast를 수행할 pendingIntent 선언
            val pendingIntent = PendingIntent.getBroadcast(
                context,
                UUID.randomUUID().variant(), // unique한 아이디 생성
                alarmIntent,
                PendingIntent.FLAG_IMMUTABLE
            )
          
            // alarmManager에 알람 등록
            alarmManager.setExact(
                AlarmManager.RTC, // 밑 알람 유형 참고
                alarmTime.time - TimeUnit.MINUTES.toMillis(10), // 해당 시간 10분 전
                pendingIntent
            )
        }
    }
```

<br>

## + 등록된 알람 취소하기

구글링을 해본 결과, alarmManager에 등록된 전체 알람을 가져올 방법은 없다. 따라서 <sapn style = "background-color:#fff5b1">등록된 알람을 취소하고자 한다면 등록할 때 만든 것과 동일한 pendingIntent를 만들어 하나하나 취소해야 한다.</span>

```kotlin
    fun cancelAllAlarm(context: Context) {
        val alarmManager = context.getSystemService(Context.ALARM_SERVICE) as AlarmManager
        //필자는 해당 pendingIntent를 따로 변수에 저장해두었다 취소할 때 사용하였다.
        pendingIntentList.forEach {
            alarmManager.cancel(it)
        }
    }
```

<br>

## 👩🏻‍💻 [알람 유형](https://developer.android.com/training/scheduling/alarms?hl=ko)

1. `ELAPSED_REALTIME`
  * <span style = "background-color:#fff5b1">기기가 부팅된 후 경과한 시간에 대기 중인 인텐트 실행</span>
  * 기기의 절전 모드 해제하지 않음
2. `ELAPSED_REALTIME_WAKEUP`
  * ELAPSED_REALTIME과 동일
  * 기기의 절전 모드를 해제하고 대기 중인 인텐트 실행
3. `RTC`
  * <span style = "background-color:#fff5b1">지정된 시간에 대기 중인 인텐트 실행</span>
  * 기기의 절전 모드 해제하지 않음
4. `RTC_WAKEUP`
  * RTC와 동일
  * 기기의 절전 모드를 해제하고 대기 중인 인텐트 실행

<br>

## 👩🏻‍💻 [broadcast receiver 사용 시, 주의할 점](https://stackoverflow.com/questions/17906037/broadcastreceiver-onreceive-open-dialog)

주의할 점은 <span style = "background-color:#fff5b1">broadcast receiver를 통해 알람을 받고 UI 작업을 할 수 없다는 것이다.</span> 따라서, dialog를 띄우거나 할 수 없다.. 하지만 pendingIntent를 활용해 activity를 다이얼로그처럼 띄울 수 있다. 이 방법은 추후 별도 게시글을 통해 정리해보겠다.

+ broadcast를 사용한다면 다음 게시글을 통해 해당 [사용 권장사항](https://developer.android.com/guide/components/broadcasts?hl=ko#security-and-best-practices)을 읽어보는 것이 좋을 것 같다!

## 👩🏻‍💻 참고
* <https://velog.io/@thevlakk/Android-AlarmManager-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-1>