---
title: "[Android] navigation safe args 사용하기"
excerpt: "fragment간 데이터 전달"
categories:
  - Android
tag:
  - navigation safe args
  - safe args

last_modified_at: 2023-03-19
toc: true
toc_sticky: true
search: true
---

# 🙋‍♀️ safe args?
  기존 fragment간 데이터를 전달할 때는 bundle을 사용했다. 하지만 bundle의 경우, **타입 안정성을 보장하지 않는다**는 한계가 존재했다.

  그래서 등장한 것이 navigation의 **safe args**이다!!

<br>

# 🙋‍♀️ 사용법?

사용법도 비교적 간단하니 빠르게 살펴보자!!

## 👀 필요한 플러그인 추가

```kotlin
//app level gradle
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
    id 'com.google.android.libraries.mapsplatform.secrets-gradle-plugin'
    id 'androidx.navigation.safeargs.kotlin' //⭐⭐⭐
}
```

```kotlin
//project level gradle
plugins {
    id 'com.android.application' version '7.3.1' apply false
    id 'com.android.library' version '7.3.1' apply false
    id 'org.jetbrains.kotlin.android' version '1.7.20' apply false
    id 'com.google.android.libraries.mapsplatform.secrets-gradle-plugin' version '2.0.1' apply false
    id 'androidx.navigation.safeargs.kotlin' version '2.5.3' apply false //⭐⭐⭐ 버전은 검색을 통해 최신 버전을 사용하면 된다!
}
```

<br>

## 👀 Parcelable 사용하기
데이터를 safe args를 사용해 전달할 때 정렬화가 이루어져야 한다.

이를 위해서는 **Percelable**을 사용한다. 이는 기존의 Serializable보다 빠르고, boiler plate 코드가 비교적 적다.

```kotlin
//app level gradle
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
    id 'com.google.android.libraries.mapsplatform.secrets-gradle-plugin'
    id 'androidx.navigation.safeargs.kotlin'
    id 'kotlin-parcelize'//⭐⭐⭐
}
```

이제 전달하고자 하는 데이터 객체에 Percelable을 적용하면 된다!

```kotlin
@Parcelize
data class Person(
    val uuid: Int?,
    val favorite: Boolean,
    ...
    val relation: String,
    val birth: LocalDateTime,
    val residence: String,
    val score: Int?
): Parcelable
```

그렇다면 전달하기 위한 준비는 끝났다!! 이제는 직접 전달하고 값을 가져와 사용해보자!

## 👀 값 전달 및 사용

우선 navigation에 값이 전달될 fragment에 arguments를 등록해준다!

<img src = "https://drive.google.com/uc?id=1aofuj7NVVIuTp9YFKL9eLb5tmWTITQUL" width = 600 height = 500>

위와 같이 데이터 값의 이름을 정의하면 코드상에서 해당 이름을 통해 값에 접근할 수 있게 된다!

```kotlin
//값 전달
        peopleListAdapter.setOnItemClickListener { person ->
            val action = PeopleListFragmentDirections.actionFragmentPeopleListToFragmentPersonDetail(it)
            findNavController().navigate(action)
        }
```

```kotlin
//값 사용
      private val args: PersonDetailFragmentArgs by navArgs<PersonDetailFragmentArgs>() //args 선언

      personDetailViewModel.setPerson(args.person) // navGraph에서 정의한 이름으로 사용
```

**🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!**

<br>

## 📃참고
* 공식 문서
    * <https://developer.android.com/reference/android/speech/SpeechRecognizer>
    * <https://developer.android.com/reference/android/speech/RecognizerIntent>
    * <https://developer.android.com/reference/android/speech/RecognitionListener>
* 그 외
    * <https://naver.github.io/naverspeech-sdk-android/index.html?com/naver/speech/clientapi/SpeechRecognizer.html>
    * <https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=horajjan&logNo=220329237271>
    * <https://ebbnflow.tistory.com/188>