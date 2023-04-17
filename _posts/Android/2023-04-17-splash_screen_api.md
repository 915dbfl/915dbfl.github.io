---
title: "[Android] Splash Screen Api 적용하기"
excerpt: "android 12 이상 splash screen 대응하기"
categories:
  - Android
tag:
  - splash screen api

last_modified_at: 2023-04-17
toc: true
toc_sticky: true
search: true
---

# 🙋‍♀️ Splash Screen Api?

**안드로이드 12**를 시작으로 splash screen을 적용하는 방식에 변경이 생겼다. 안드로이드 12부터는 **기본 splash 화면**이 적용되며, 이는 앱 런처 아이콘 요소 및 테마 windowBackground를 사용하여 구성이 된다고 한다. 


그렇다면 기존에는 어떠한 방식으로 splash 화면을 제작했을까?

1. `android:windowBackground`를 재정의하는 맞춤 테마를 사용해 구현
    * android 12 이상에서는 기본 android 시스템 splash 화면으로 대체가 되어 의도하지 않은 결과가 초래될 수 있다.
2. splash를 위한 activity를 별도로 생성해 사용
    * android 12 이상에서는 **기본 splash 화면 이후, activity로 설정한 splash 화면이 중복으로 출력**되게 된다.

그렇다면 왜 splash api가 등장하게 되었을까?🤔
* 기존 방식대로라면 앱이 금방 다시 실행되는 hot-start 상태에서 **이미 로드된 리소스들이 다시 로드되게 된다.** 
* 사용자 입장에서는 불필요한 앱의 대기 시간이 생기는 것이다.

따라서 안드로이드 12에서는 앱의 상태에 따라 splash를 제어해주는 splash api가 등장하게 되었다. 그럼 본격적으로 그 사용법에 대해 알아보자!

<br>

# 🙋‍♀️ 사용법

## ✓ 의존성 추가
```kotlin
    //gradle: module
    implementation 'androidx.core:core-splashscreen:1.0.0'
```

## ✓ theme 정의 
```kotlin
    <style name="Theme.Sopar.Splash" parent="Theme.SplashScreen">
        <item name="windowSplashScreenBackground">@color/white</item>
        <item name="windowSplashScreenAnimatedIcon">@drawable/ic_sopar_splash_logo</item>
        <item name="windowSplashScreenAnimationDuration">1000</item>
        <item name="postSplashScreenTheme">@style/Theme.Sopar</item>
    </style>
```
* `windowSplashScreenBackground`: 스플래시 화면의 배경 색 지정
* `windowSplashScreenAnimatedIcon`: 스플래시 화면의 중앙 아이콘 지정
* `windowSplashScreenAnimationDuration`: 스플래시 화면의 지속 시간 지정
* `postSplashScreenTheme`: 스플래시 화면 후에 보일 화면 테마 지정

이 외에도 다른 속성을 알고 싶다면 [공식 문서](https://developer.android.com/guide/topics/ui/splash-screen?authuser=1&hl=ko)를 참고하자!

## ✓ manifest에 splash theme 적용
```kotlin
   <application
        android:name=".presentation.base.GlobalApplication"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.Sopar.Splash" // ⭐️
        tools:targetApi="31">
```
* application에 정의한 splash theme을 적용해준다.

## ✓ activity 코드 수정
```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        installSplashScreen() // ⭐️
        ...
    }
```
* installSplashScreen()
    ```kotlin
        @JvmStatic
        public fun Activity.installSplashScreen(): SplashScreen {
            val splashScreen = SplashScreen(this)
            splashScreen.install()
            return splashScreen
        }
    ```
    * 스플래시 화면 객체를 반환받아 애니메이션을 맞춤설정하거나 splash 화면을 더 오래 표시할 수 있다.

## ✓ splash 화면 닫기 애니메이션 맞춤설정
```kotlin
        this.splashScreen.setOnExitAnimationListener { splashScreenView ->
            val slideUp = ObjectAnimator.ofFloat(
                splashScreenView,
                View.TRANSLATION_Y,
                0f,
                -splashScreenView.height.toFloat()
            )
            slideUp.interpolator = AnticipateInterpolator()
            slideUp.duration = 200L

            slideUp.doOnEnd { splashScreenView.remove() }
            slideUp.start()
        }
```
* [ObjectAnimator](https://developer.android.com/guide/topics/graphics/prop-animation?hl=ko): 애니메이션으로 보여줄 타겟 객체 및 객체 속성을 설정할 때 사용된다.
    * ~~해당 객체말고 안드로이드에서는 다양한 상황에 커스텀 애니메이션을 적용할 수 있는 여러 객체를 제공한다. 위 공식 문서를 통해 추후 학습을 진행해봐야겠다.~~


**🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!**

<br>

## 📃참고
* 공식 문서
    * <https://developer.android.com/guide/topics/ui/splash-screen?hl=ko>
    * <https://developer.android.com/guide/topics/ui/splash-screen/migrate?hl=ko>
* 그 외
    * <https://sungbin.land/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C12-%EC%8A%A4%ED%94%8C%EB%9E%98%EC%8B%9C-%EB%8C%80%EC%9D%91%ED%95%98%EA%B8%B0-1729f69dc33f>
    * <https://yoon-dailylife.tistory.com/124>
    * <https://velog.io/@pachuho/Android-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-12-Splash-Screen-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0>