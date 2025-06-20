---
title: "[Android] 반복되는 빌드 로직을 어떻게 처리할 수 있을까?"
excerpt: "Gradle Plugin이란 무엇일까? 적용에 필요한 사전 지식을 차근히 살펴보자!"
categories:
  - Android
tag:
  - Android
  - gradle plugin
  - script plugin
  - binary plugin
  - precompiled script plugin
  - composite build
  - parallel build
  - build cache

last_modified_at: 2025-06-20
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

<br>

# 🔗 buildSrc vs build-logic

gradle plugin을 구성하다 보면 `buildSrc`를 활용하는 코드와 `build-logic` 을 활용하는 코드, 두 가지를 모두 볼 수 있다. (공식 문서에서 Binary / Precompiled Script Plugin을 적용하는 예시를 본다면 buildSrc가 함께 사용된다.)

기존에는 `buildSrc`를 많이 사용하였다. 하지만 `nowinandroid`, `droidknights` 코드를 확인해본다면 plugin을 구성하는 방식에는 차이가 존재하지만 결국 `build-logic`을 활용하고 있는 것을 확인할 수 있다. <span style = "background-color:#fff5b1">그렇다면 왜 buildSrc라는 편리한 방식이 존재함에도 불구하고 별도의 모듈을 생성하는 것일까?</span>

## 1. buildSrc란?

buildSrc는 dependency 및 build logic을 관리하는 하나의 기술 중 하나이다. 이는 별도의 외부 파일을 생성하지 않고, `buildSrc` 폴더를 프로젝트 내 만들어 손쉽게 활용할 수 있다.

buildSrc 내부에는 크게 다음의 작업이 가능하다.

1. custom gradle plugin 위치
2. dependency 정의 및 관리
3. build logic 캡슐화

하지만 buildSrc의 경우, <span style = "background-color:#fff5b1">한 줄을 수정할 경우 프로젝트 캐시를 무효화 할 가능성 이 있다.</span> 그렇다면 내부적으로 어떻게 동작하길래 한 줄의 수정이 프로젝트 전체 캐시에 영향을 주는 것일까?

buildSrc를 사용하게 된다면 composite build 형태로 buildSrc 내부 로직이 우선 빌드가 된다. 그리고 이렇게 우선 빌드된 결과들이 실제 사용 위치와 상관없이 <span style = "background-color:#fff5b1">프로젝트의 모든 모듈 속 build 스크립트의 클래스 패스에 추가가 되게 된다.</span> 그렇기 때문에 별도의 처리 없이도 모든 모듈에서 buildSrc 로직에 접근해 활용할 수 있게 된다.

따라서 buildSrc를 사용할 경우, 자동적으로 모든 모듈에서 buildSrc 로직에 대한 의존성을 가지기 때문에 변경사항이 프로젝트 전체에 영향을 미치게 된다.

## 2. 별도의 모듈 생성 + composite build 설정

그렇다면 build-logic과 같이 별도의 모듈을 생성해 직접 `composite build`로 설정하게 된다면 어떻게 될까? 직접 composite build로 설정을 해준다면 <span style = "background-color:#fff5b1">사용하는 곳에만 클래스 패스가 추가되게 된다.</span> 그렇기 때문에 buildSrc와 달리 빌드 로직의 수정이 사용하는 곳에만 영향을 주게 된다.

따라서 buildSrc를 사용하지 않고 별도의 모듈을 활용해 composite build를 적용하게 된다면 불필요하게 프로젝트 캐시가 무효화되는 문제를 예방할 수 있다.


## ➕ 여러 프로젝트에서 빌드 로직을 공유하고 싶다면? build-logic!

나아가 <span style = "background-color:#fff5b1">여러 프로젝트에서 빌드 로직을 공유하고 싶을 때도 별도의 모듈을 선언해 활용해야 한다.</span>

![buildSrc](/assets/images/buildsrc.png)

buildSrc의 경우, 프로젝트마다 하나씩 존재하기 때문에 buildSrc를 사용해 빌드 로직을 공유할 수 없다.

![build logic](/assets/images/build_logic.png)

build-logic 모듈을 정의 composite build로 적용하게 된다면 특정 모듈에 종속되는 빌드 로직이 아니다 보니 여러 프로젝트에서 공유하는 build logic을 구성할 수 있게 된다.

| buildSrc | build-logic |
| --- | --- |
| 단일 프로젝트 추천 | 다중 프로젝트 및 레포지토리 추천 |
| 자주 변하지 않는 값(ex: 앱 버전) 저장 | 빌드 로직을 루트 프로젝트에 종속성이 없도록 만들고자 할 때 / 빌드 로직을 뗐다 붙였다 하고 싶을 때 |

<br>

# 🔗 Composite Build의 장단점

Composite Build란 독립적으로 개발된 다른 gradle build를 현재 빌드에 포함시키는 역할을 한다. plugin을 정의하는 방식 중 `Binary / Precompiled`를 활용할 경우, 별도의 모듈을 선언해 적용하게 된다면 아래와 같이 composite build 설정을 진행해줘야 한다. 

```gradle
pluginManagement {
    includeBuild("build-logic") // composite build 설정
    repositories {
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
    }
}
```

이처럼 composite build를 사용하게 된다면 프로젝트 빌드보다 해당 모듈의 빌드 로직이 우선적으로 진행되게 된다. 이러한 특성으로 다음과 같은 장단점이 발생하게 된다.

- 장점
    - Binary / Precompiled Script Plugin을 적용하게 된다면 빌드 로직이 우선 빌드돼 `JAR` 형태로 패키징이 되고, 추후 불필요한 재빌드 없이 해당 빌드 결과를 재활용할 수 있게 된다.
- 단점
    - 프로젝트 빌드 이전에 우선 빌드가 진행되기 때문에 <span style = "background-color:#fff5b1">초기 빌드 시간이 늘어나게 된다.</span>

모든 기술에는 장단점이 존재하기 때문에 이를 명확히 인식할 필요가 있다. Binary / Precompiled Script Plugin을 적용하면 중복 빌드 로직을 제거하고, 빌드 로직의 재사용성을 높일 수 있다는 장점이 존재한다. 하지만 빌드 로직의 재사용성을 높이는 대신 초기 빌드 시간이 늘어난다는 단점 역시 존재한다. 따라서 이러한 늘어나는 초기 빌드 시간을 줄이기 위해 추가적인 노력이 필요하다.

<br>

# 🔗 빌드 시간 최소화

gradle에서는 빌드 시간을 최소화하기 위해 적용할 수 있는 다양한 방식이 존재한다. 대표적으로가 `병렬 빌드`와 `gradle build cache`를 활용하는 방식이다.

## 1. parallel build

병렬 빌드를 이해하기 위해서는 gradle의 빌드 lifecycle에 대한 이해가 필요하다. 만약 해당 부분에 대한 이해가 필요하다면 다음 게시글의 Gradle Part만 읽어보고 오기를 추천한다. ([[Build] build 그리고 build tool: Gradle](https://915dbfl.github.io/android/build-and-build-tool/))

Gradle은 크게 초기화 -> 구성 -> 실행 단계를 거치게 된다. 그리고 실행 단계에서는 구성 단계에서 만든 `task graph`를 활용하게 된다. graph를 참고해 task들의 의존성에 따라 실행이 진행되는 것이다.

parallel build에서는 이러한 task graph에서 <span style = "background-color:#fff5b1">의존하지 않는 서로 다른 task들을 병렬적으로 실행하도록 한다.</span> 따라서 자연스럽게 빌드 시간이 단축되는 것이다.

## 2. gradle build cache

gradle build cache의 경우에는 <span style = "background-color:#fff5b1">이전 빌드 태스크의 결과를 가져와 재사용하기 위해 사용된다. 이는 주로 `checkout`이나 `초기 빌드`와 같이 gradle의 증분 빌드가 제대로 적용되지 못하는 상황에서 빌드 속도를 향상시킬 수 있다.

`증분 빌드`라는 것은 gradle에서 기본적으로 제공해주는 기능으로 변경사항이 발생했을 때 그와 관련된 task들만 효과적으로 재빌드하는 과정이다. 이때는 task의 input 달라졌을 때만 해당 task를 재빌드하고, input이 동일하다면 이전 빌드에서의 task 결과를 재사용하게 되는 것이다.

<span style = "background-color:#fff5b1">checkout이나 초기 빌드 상황에서는 이전 빌드가 존재하지 않기 때문에 증분 빌드를 활용할 수 없다. 이러한 상황에서 gradle build cache를 사용한다면 빌드에서의 task 결과값이 저장되어 증분 빌드가 활용될 수 없는 상황에서도 빌드 속도의 효율을 높일 수 있다.</span>

<br>

# 🔗 Plugin 적용하기

관련해서는 별도의 게시글을 작성하였다. 전반적인 과정을 중심으로 다뤘으니, 세부적인 코드는 해당 Repo의 build-logic 및 gradle 파일들을 참고하기를 바란다.(모듈별 gradle 파일, settings.gradle 파일)

> [[Android] gradle plugin 활용하기](https://915dbfl.github.io/android/gradle-plugin(2)/)

<br>

# 🔗 참고 자료
- [https://www.youtube.com/watch?v=7iag6zpGd98](https://www.youtube.com/watch?v=7iag6zpGd98)
- [https://www.slideshare.net/slideshow/gradle-kotlin/261178195#77](https://www.slideshare.net/slideshow/gradle-kotlin/261178195#77)
- [https://github.com/droidknights/DroidKnightsApp/blob/main/build-logic/src/main/kotlin/com/droidknights/app/KotlinAndroid.kt](https://github.com/droidknights/DroidKnightsApp/blob/main/build-logic/src/main/kotlin/com/droidknights/app/KotlinAndroid.kt)
- [https://github.com/android/nowinandroid](https://github.com/android/nowinandroid)