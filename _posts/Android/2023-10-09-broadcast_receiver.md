---
title: "[Broadcast Receiver] 알람 후 다이얼로그를 열려면?"
excerpt: "activity를 마치 다이얼로그처럼"
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

## 👩🏻‍💻 Broadcast Receiver

broadcast receiver 작업은 백그라운드 작업이기 때문에 context를 필요로 하는 dialog를 직접적으로 띄울 순 없다. 따라서 우리는 <span style = "background-color:#fff5b1">pendingIntent</span>를 적절히 사용해볼 것이다.

## 👩🏻‍💻 activity를 다이얼로그처럼!

1. 우선 띄울 다이얼로그를 activity class로 정의한다.
2. activity를 manifest에 등록한다.
  ```kotlin
          <activity
            android:name=".common_ui.presenter.alarm.PreorderAlarmDialog"
            android:exported="false"
            android:theme="@style/Theme.DialogTheme" />
  ```
  * 이때 acitivty에 dialog 테마를 적용한다.
    - 원하는대로 커스텀을 진행하면 된다.

      ```kotlin
          <style name="Theme.DialogTheme" parent="Theme.MaterialComponents.NoActionBar">
          <item name="android:layout_width">wrap_content</item>
          <item name="android:layout_height">wrap_content</item>
          <item name="android:layout_gravity">end|top</item>
          <item name="android:windowBackground">@android:color/transparent</item>
          <item name="android:colorBackgroundCacheHint">@null</item>
          <item name="android:windowIsTranslucent">true</item>
          <item name="android:windowAnimationStyle">@android:style/Animation</item>
          <item name="android:windowNoTitle">true</item>
          <item name="android:windowContentOverlay">@null</item>
          <item name="android:windowFullscreen">true</item>
      </style>
      ```
3. broadcast receiver에서 activity를 실행하는 pendingIntent를 생성한다.

  ```kotlin
  class AlarmReceiver : BroadcastReceiver() {
    override fun onReceive(p0: Context?, p1: Intent?) {
        val intent = Intent(p0, PreorderAlarmDialog::class.java)
        PendingIntent.getActivity(p0, 0, intent, PendingIntent.FLAG_IMMUTABLE).send()
    }
  } 
  ```

이렇게 하면 액티비티가 마치 다이얼로그 처럼 뜨는 것을 확인할 수 있을 것이다.

<br>

## 👩🏻‍💻 + activity inner class로 broadcast receiver 정의하기

* <https://stackoverflow.com/questions/25215878/how-to-update-the-ui-of-activity-from-broadcastreceiver>

activity 내부에 broadcast receiver를 정의해 context에 접근하지 않고도 ui를 업데이트할 수 있는 방식도 있다.

<br>

## 👩🏻‍💻 참고
* <https://blog.naver.com/yjsplay2002/50107917483>