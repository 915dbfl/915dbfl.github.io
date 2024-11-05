---
title: "[Android] gradle plugin 활용하기(1)"
excerpt: "반복되는 build logic 어떻게 처리할 수 있을까? / Script Plugin 적용해보기!"
categories:
  - Android
tag:
  - Android
  - gradle plugin
  - script plugin
  - binary plugin
  - precompiled script plugin
  - build-logic

last_modified_at: 2024-11-05
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며: 생성된 build.gradle.kts 파일을 그냥 쓰면 안될까?

멀티 모듈을 생성하게 되면 모듈마다 하나의 `build.gradle.kts` 파일이 생기게 되는데, 내부를 살펴보면 다음과 같이 중복되는 코드를 확인할 수 있다.

```kotlin
plugins {
    alias(libs.plugins.android.library)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.kotlin.serialization)
}

android {
    namespace = "kr.boostcamp_2024.course.login"
    compileSdk = 34

    defaultConfig {
        minSdk = 24

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        consumerProguardFiles("consumer-rules.pro")
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
}
```

모듈이 점점 많아진다면, 위와 같이 반복되는 configuration code가 많아지게 된다. 이렇게 중복되는 코드가 많아진다는 것은 결국 하나의 변경사항이 생겼을 때 모든 모듈의 build.gradle.kts 파일을 수정해야 한다는 것을 의미한다.

이러한 상황에서 `gradle plugin`을 적용할 수 있다. gradle plugin 적용 방식은 크게 다음 세가지가 존재한다.

1. script plugin
2. binary plugin
3. precompiled script plugin

# 🔗 Gradle Plugin

## 1) gradle plugin이 뭔데?

- 빌드 로직을 재사용할 수 있는 확장 기능을 제공한다.
- gradle task를 추가 or 관리할 수 있다.

우리는 이미 `gradle plugin`을 활용하고 있다. 어디서 적용되고 있을까?

```kotlin
id("com.android.application")
// 플러그인이 없다면 수많은 중복코드 생성해야 함
```

## 2) 각각의 적용 방식 간단히 확인해보기!

### 1️⃣ script plugin

- 단순히 별도의 gradle 파일로 분리해 apply하는 방식 ( = common.gradle 파일 구성)

```kotlin
apply(from = "common.gradle")
```
        
### 2️⃣ binary plugin
- Pluing 인터페이스 구현 클래스 작성
- 플러그인 아이디를 등록해서 사용

```kotlin
plugins {
    id("my.custom.plugin")
}
```
        
### 3️⃣ precompiled script plugin
- script + binary 특징을 모두 가지는 방법
- 클래스를 직접 작성하는 대신 `xxx.gradle.kts` 파일을 만들어 스크립트 형태로 간편하게 확장 플러그인 생성

```kotlin
// common.android.gradle.kts 존재
// build.gradle.kts
plugins {
    id("common.android")
}
```

## 3) buildSrc vs build-logic

gradle plugin을 구성하다 보면 `buildSrc`를 활용하는 코드와 `build-logic` 을 활용하는 코드, 두 가지를 모두 볼 수 있다.

기존에는 `buildSrc`를 많이 사용하였다. 하지만 `nowinandroid`, `droidknights` 코드를 확인해본다면 plugin을 구성하는 방식에는 차이가 존재하지만 결국 `build-logic`을 활용하고 있는 것을 확인할 수 있다.

### build-logic, buildSrc와 비슷하지 않을까?

buildSrc는 dependency 및 build logic을 관리하는 하나의 기술 중 하나이다. 이는 별도의 외부 파일을 생성하지 않고, `buildSrc` 폴더를 프로젝트 내 만들어 손쉽게 활용할 수 있다.

buildSrc 내부에는 크게 다음의 작업이 가능하다.

1. custom gradle plugin 위치
2. dependency 정의 및 관리
3. build logic 캡슐화

하지만 buildSrc의 경우, <span style = "backgroud-color:#fff5b1">한 줄을 수정할 경우 buildSrc 폴더 내 gradle 파일 전체를 재확인하는 작업을 진행한다.</span> 하지만 `build-logic` 의 경우 증분 옵션을 제공해 수정한 부분만 빌드 반영이 가능해진다.

### ✔️ 여러 프로젝트에서 빌드 로직을 공유하고 싶다면? build-logic!

![buildSrc](/assets/images/buildsrc.png)

![build logic](/assets/images/build_logic.png)

- buildSrc의 경우, 프로젝트마다 하나씩 존재하기 때문에 buildSrc를 사용해 빌드 로직을 공유할 수 없다.
- build-logic 모듈을 활용해 Composite build를 적용해 여러 프로젝트에서 공유하는 build logic을 구성할 수 있게 된다.

| buildSrc | build-logic |
| --- | --- |
| 단일 프로젝트 추천 | 다중 프로젝트 및 레포지토리 추천 |
| 자주 변하지 않는 값(ex: 앱 버전) 저장 | 빌드 로직을 루트 프로젝트에 종속성이 없도록 만들고자 할 때 / 빌드 로직을 뗐다 붙였다 하고 싶을 때 |

<br>

# 🔗 Script Plugin 적용해보기

## 1. 생성

중복되는 gradle 코드를 별도의 `common.gradle` 파일로 옮겨 groovy 파일을 생성하자.

```kotlin
// 진행했던 LGTM 프로젝트 common.gradle 파일
def hasLibraryPlugin = pluginManager.hasPlugin("com.android.library")
def hasApplicationPlugin = pluginManager.hasPlugin("com.android.application")

if (hasLibraryPlugin || hasApplicationPlugin) {
    android {
        compileSdk = libs.versions.compileSdk.get().toInteger()
        defaultConfig {
            minSdk = libs.versions.minSdk.get().toInteger()

            if (hasLibraryPlugin) consumerProguardFiles("consumer-rules.pro")
            testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        }

        compileOptions {
            sourceCompatibility = JavaVersion.VERSION_17
            targetCompatibility = JavaVersion.VERSION_17
        }

        kotlinOptions {
            jvmTarget = '17'
        }
    }
}
```

build script 설정에서는 `plugins{}` 를 바로 사용할 수 없다!
→ 따라서 플러그인을 적용하기 위해 예전에 사용했던 방식인 `apply`를 적용해야 한다.

```kotlin
apply plugin: 'com.google.dagger.hilt.android'
```

## 2. 적용

```kotlin
// build.gradle.kts :feature:main

plugins {
	//..
}

apply(from = "common.android.gradle")

dependencies {
	//...
}
```

script 로직은 `inline` 으로 포함되어 동작한다. 따라서 `apply`를 통해 함수를 사용하듯이 적용할 수 있다. 또한, 중복되는 script를 모아 하나의 script로 제공 가능하다.

```kotlin
// feature 모듈에 적용되는 script를 하나의 파일에 모음
// common.android.feature.gradle

apply(from = "common.android.gradle")
apply(from = "common.android.compose.gradle")
//..

dependecies {
	impelemtation(project(":core:domain"))
	impelemtation(project(":core:designsystem"))
	//..
}
```

```kotlin
// 사용
apply(from = "coommon.android.feature.gradle")
```

## 3. 특징

적용해보니 어떤가? 그냥 단순히 중복되는 로직을 별도의 `gradle` 파일에 배치하기만 하면 된다.

- 가장 단순하고 빠르게 적용할 수 있다.
- 왜 `groovy`로만 작성을 해야 할까?
    - `gradle kotlin dsl`은 타입 안전성 제공 → `build.gradle`이라는 파일을 인식 후 빌드
    - `xxxx.gradle`  형태의 별도의 gradle 파일을 인식하지 못함

# 🔗 마무리하며

`Script Plugin`을 적용하게 된다면 모듈 속 `build.gradle` 파일의 로직들이 꽤 깔끔해진 것을 확인할 수 있다. 해당 방식은 예전에 많이 사용되었던 방식이다. 로직을 캡슐화하고, 재사용 가능하도록 하는 더 나은 방식 `Binary Plugin`, `PreCompiled Script Plugin`에 대해서도 다음 게시글에서 다뤄보자!

# 🔗 참고 자료
- [https://www.youtube.com/watch?v=7iag6zpGd98](https://www.youtube.com/watch?v=7iag6zpGd98)
- [https://www.slideshare.net/slideshow/gradle-kotlin/261178195#77](https://www.slideshare.net/slideshow/gradle-kotlin/261178195#77)
- [https://github.com/droidknights/DroidKnightsApp/blob/main/build-logic/src/main/kotlin/com/droidknights/app/KotlinAndroid.kt](https://github.com/droidknights/DroidKnightsApp/blob/main/build-logic/src/main/kotlin/com/droidknights/app/KotlinAndroid.kt)
- [https://github.com/android/nowinandroid](https://github.com/android/nowinandroid)