---
title: "[Android] API key 숨기기"
excerpt: "local properties 이용하기"
categories:
  - Android
tag:
  - local properties
  - Secrets Gradle Plugin for Android

last_modified_at: 2023-04-15
toc: true
toc_sticky: true
search: true
---

# 🙋‍♀️ Secretes Gradle Plugin for Android

다양한 api를 활용해 개발을 하다 보면 api key 노출을 막아야 한다.

이때 우리가 사용할 수 있는 것이 바로 `Secrets Gradle Plugin for Android`이다. 우리는 이를 통해 **안전하게 필요한 키를 주입할 수 있게 된다.**

# 🙋‍♀️ 사용법?

## ✓ gradle 추가

우선 필요한 플러그인을 gradle에 추가하자!

```kotlin
// gradle: project(루트)
plugins {
    id 'com.android.application' version '7.3.1' apply false
    id 'com.android.library' version '7.3.1' apply false
    id 'org.jetbrains.kotlin.android' version '1.7.20' apply false
    id 'com.google.android.libraries.mapsplatform.secrets-gradle-plugin' version '2.0.1' apply false // ⭐⭐⭐
}
```

```kotlin
//gradle: module(앱)
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
    id 'com.google.android.libraries.mapsplatform.secrets-gradle-plugin' // ⭐⭐⭐
}
```

## ✓ local.properties 사용하기
* 로컬 컴퓨터의 **환경**을 저장하는 파일
* 버전 관리에서 제외됨으로써 파일 속 값들이 노출되지 않는다!

이제 api key 값을 `local.properties`에 저장해보자!

```kotlin
bookApiKey=...
```

위와 같이 api key를 작성했다면 다음 코드를 통해 이를 활용할 수 있다.

```kotlin
object Constants {
    const val BASE_VUL = "https://dapi.kakao.com/"
    const val API_KEY = BuildConfig.bookApiKey //⭐⭐⭐
}
```

## ✓ 매니페스트에서 빌드 변수가 필요하다면??

```kotlin
plugins {
...
    id 'com.google.android.libraries.mapsplatform.secrets-gradle-plugin'
}

//local.properties 가져오기
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())

android {
    namespace 'org.sopar'
    compileSdk 33

    defaultConfig {
        ...

        //키 - 값 쌍의맵으로 매니페스트에서 사용할 빌드 변수 삽입
        manifestPlaceholders["NATIVE_APP_KEY"] = properties['NATIVE_APP_KEY']
    }
```
* `manifestPlaceholders` 속성 사용

위와 같이 빌드 변수를 삽입했다면 아래와 같이 사용할 수 있게 된다.

```kotlin
            <intent-filter>
                ...
                <data android:host="oauth"
                    android:scheme="kakao${NATIVE_APP_KEY}" />
            </intent-filter>
```

**🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!**

<br>

## 📃참고
* <https://developers.kakao.com/docs/latest/ko/kakaologin/android#before-you-begin>
* <https://velog.io/@hoyaho/HideAPIKey>
* <https://developer.android.com/studio/build/manifest-build-variables?hl=ko>
* <https://kkh0977.tistory.com/306>