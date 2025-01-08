---
title: "[Android] gradle plugin 활용하기(2)"
excerpt: "Binary Plugin / Precompiled script plugin를 적용해보자!"
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

## 🔗 들어가며: gradle plugin이 무엇이며, 어떤 방식들이 존재할까?

이전 게시글에서는 `왜 gradle plugin이 필요하며`, gradle plugin을 구성하기 위해서는 어떤 방식들이 존재하는지에 다뤘다.

단순히 공통되는 빌드 로직을 별도의 gradle 파일로 구성하는 `Script Plugin` 방식에 대해서는 저번 게시글에서 다뤘다.

이번 게시글에서는 추가적으로 `Binary Plugin`, `Precompiled Script Plugin`을 적용하는 방식에 대해 깊이 다뤄보고자 한다!


## 🔗 Binary Plugin 적용해보기

해당 방식은 위의 `Script Plugin`과는 다르게 `kotlin`을 활용해 플러그인을 구성할 수 있다. `nowinandroid` 프로젝트를 보면 `build-logic` 내부에 해당 `Binary Plugin`이 적용된 것을 확인할 수 있다.

해당 방식은 플러그인을 공개된 저장소에 배포 / 다른 프로젝트에서 재사용할 경우, 가장 많이 소개되는 방식이다. 여기서는 `buildSrc`에 `Binary Plugin`을 구성하는 방식을 다뤄보겠다.

## 1. 적용

### 1️⃣ `kotlin-dsl` 플러그인 추가하기

```kotlin
// buildSrc/build.gradle.kts
plubins {
	`kotlin-dsl`
}
// buildSrc에 필요한 의존성들 추가
dependencies {
	implementation(libs.android.gradlePlugin)
	implementation(libs.kotlin.gradlePlugin)
	implementation(libs.hilt.gradlePlugin)
}
```

### 2️⃣ 필요한 플러그인 생성

```kotlin
// nowinandroid: build-logic/convention/src/main/kotlin/AndroidLibraryConventionPlugin.kt
class AndroidLibraryConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("com.android.library")
                apply("org.jetbrains.kotlin.android")
                apply("nowinandroid.android.lint")
            }

            extensions.configure<LibraryExtension> {
                configureKotlinAndroid(this)
                defaultConfig.targetSdk = 34
                defaultConfig.testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
                testOptions.animationsDisabled = true
                configureFlavors(this)
                configureGradleManagedDevices(this)
                // The resource prefix is derived from the module name,
                // so resources inside ":core:module1" must be prefixed with "core_module1_"
                resourcePrefix = path.split("""\W""".toRegex()).drop(1).distinct().joinToString(separator = "_").lowercase() + "_"
            }
            extensions.configure<LibraryAndroidComponentsExtension> {
                configurePrintApksTask(this)
                disableUnnecessaryAndroidTests(target)
            }
            dependencies {
                add("androidTestImplementation", kotlin("test"))
                add("testImplementation", kotlin("test"))

                add("implementation", libs.findLibrary("androidx.tracing.ktx").get())
            }
        }
    }
}
```

nowinandroid 속 생성된 binary plugin 코드를 가져와봤다. 코드를 우리가 평소 script를 사용하여 작업했던 것보다 살짝 복잡한 방식으로 코드가 구성되는 것을 확인할 수 있다.

이처럼 script 환경이 아니다 보니 gradle interface를 사용할 수 없으며, `apply()` 등과 같은 별도의 방식을 사용해야 한다. 또한, android { } 역시 kotlin dsl로 만들어졌기 때문에 별도의 환경인 Plugin에서는 사용할 수 없다. 따라서 해당 방식을 적용하기 위해서는 gradle / kotlin dsl에 대한 이해가 필요하다! (필자도 해당 부분에 대한 이해가 부족해 삽질을 많이 했다,,😭)

> 실제로 이런 복잡한 동작이 `kotlin dsl` 내부에서 진행되고 있다. 우리는 `kotlin dsl`을 통해서 이런 복잡한 과정 없이 편리하게 빌드 로직을 구성할 수 있었던 것이다.

### 3️⃣ 플러그인 등록

이제 생성한 플러그인을 `buildSrc`의 `build.gradle.kts` 파일에 등록해 사용할 차례이다.

```kotlin
// nowinandroid: build-logic/convention/build.gradle.kts
gradlePlugin {
    plugins {
        register("androidApplicationCompose") {
            id = "nowinandroid.android.application.compose"
            implementationClass = "AndroidApplicationComposeConventionPlugin"
        }
        register("androidApplication") {
            id = "nowinandroid.android.application"
            implementationClass = "AndroidApplicationConventionPlugin"
        }
        //..
   }
}
```
다음과 같이 플러그인을 만들 때 생성한 클래스 파일과 실제 우리가 사용할 id를 연결해주어야 한다.

### 4️⃣ 플러그인 사용

```kotlin
// build.gradle.kts (:core:designsystem)
plugins {
	id("convention.android.library")
	id("convention.android.compose")
}
```

이제 생성한 플러그인이 필요한 모듈의 build gradle 파일에서 위와 같이 가져와 사용할 수 있다.

해당 방식 또한, 마찬가지로 필요한 플러그인들을 모아 하나의 또 다른 플러그인을 만들어 필요한 플러그인들을 한꺼번에 적용 가능하다.

```kotlin
// nowinandroid: build-logic/convention/src/main/kotlin/AndroidFeatureConventionPlugin.kt
class AndroidFeatureConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply {
                apply("nowinandroid.android.library")
                apply("nowinandroid.hilt")
                apply("org.jetbrains.kotlin.plugin.serialization")
            }
            extensions.configure<LibraryExtension> {
                testOptions.animationsDisabled = true
                configureGradleManagedDevices(this)
            }

            dependencies {
                add("implementation", project(":core:ui"))
                add("implementation", project(":core:designsystem"))
						}
			 }
	 }
}
```

```kotlin
// nowinandroid: feature/foryou/build.gradle.kts
plugins {
    alias(libs.plugins.nowinandroid.android.feature)
    //...
}
```

## 3. 특징

- 빌드 로직 캡슐화
    - script plugin보다 추상적이고 높은 확장성 제공
    - 빌드 로직 클래스로 작성 → compiler 안정성 제공
- kotlin dsl & script 코드에 대한 이해 필요
- gradle task와 같은 동작을 조합해서 복잡한 기능 사용 가능
- 빌드 로직 컨벤션을 하나의 플러그인으로 만들어 활용 가능

<br>

## 🔗 Precompiled Script Plugin 적용해보기

이제 마지막 방식이다. 위의 `Binary Plugin` 방식을 보니 어떤가? 간단한 빌드 로직을 작성해야 할 경우, 반복해서 클래스를 작성해야 한다는 것은 매우 불편한 작업이다.

해당 방식은 srcipt plugin 특징 + binary plugin 특징을 가지고 있다. 매번 클래스를 만들 필요 없이 `xxx.gradle.kts` 파일의 스크립트 형태로 간편하게 작성해 플러그인을 정의할 수 있다.

## 1. 적용

### 1️⃣ build-logic 모듈 생성, 필요한 파일 세팅

### 1. app root project setting.gradle.kts 파일 수정

build-logic 모듈을 생성하게 되면 자동으로 app 레벨의 settings.gradle.kts 파일을 수정해줘야 한다.

a. `build-logic:convention` 모듈 include 제거
- build-logic 모듈은 빌드에 사용되는 모듈이기 때문에 프로젝트에서 빌드할 필요가 없는 모듈이기 때문이다.
    
b.  pluginManagement에 `includeBuild("build-logic")`  추가
- build-logic 모듈이 빌드 모듈이라는 것을 명시

```kotlin
pluginManagement {
    includeBuild("build-logic") // 추가
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

rootProject.name = "wequiz"
include(":core:designsystem")
//...
//include(":build-logic:convention") 제거

```

------

### 2. build-logic 폴더에 파일 구성

모듈을 생성하면 기본적으로 `settings.gradle.kts`, `build.gradle.kts` 파일이 존재한다. 우리는 해당 파일들의 내용을 알맞게 수정하고, 추가로 `gradle.properties`까지 구성할 것이다.

우선 각 파일에 어떤 내용이 들어가는지 다루기 전에 각 파일이 어떤 역할을 하는지 짚고 넘어가자!

- `build.gradle.kts`
    - 모듈별로 하나씩 가지고 있음
    - 하위 subproject 당 하나의 project instance가 생성되며, project instance마다 build.gradle 파일이 실행되게 된다.
    - 크게 project, task로 구성된다.
- `settings.gradle.kts`
    - build.gradle.kts와 다르게 gradle build마다 settings.gradle.kts 파일이 실행된다.
        - multi-project build를 위한 project / task를 정의하는데 사용
    - subProject가 `include` 형태로 들어가 있음
- `gradle.properties`
    - default로 생성되지 않음, 필요에 따라 만들어야 함
    - key, value 형태로 값이 저장된다.
    - 해당 파일을 parsing하기 위해서는 JVM이 launching 되어야 한다.
    - framework 자체의 행동이나, configuration을 위한 command line flags를 사용하는 대신 해당 파일에 정의해 사용할 수 있음

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
    versionCatalogs {
        create("libs") {
            from(files("../gradle/libs.versions.toml"))
        }
    }
}

rootProject.name = "build-logic"
include(":convention")
```

```kotlin
// gradle.properties
org.gradle.parallel=true // 병렬 빌드
org.gradle.caching=true // 빌드 캐시, 이전 빌드 결과 재사용
org.gradle.configureondemand=true // 현재 태스크에 필요한 것만 빌드
```

```kotlin
// build.gradle.kts(build-logic 모듈)
plugins {
    `kotlin-dsl`
}

dependencies {
    implementation(libs.android.gradlePlugin)
    implementation(libs.android.desugarJdkLibs)
    implementation(libs.kotlin.gradlePlugin)
    compileOnly(libs.compose.compiler.gradle.plugin)
}
```

`org.gradle.configureondemand=true`에 대한 추가 설명을 조금 더 하자면, gradle의 빌드 과정은 크게 `initialization`, `configuration`, `execution`으로 진행된다. gradle build를 진행하면 모든 프로젝트에 해대 `configuration` 과정에서 각 프로젝트에 필요한 task들에 대한 task graph를 생성한다. <span style = "background-color:#fff5b1">멀티 프로젝트 빌드에서는 모든 프로젝트가 현재 실행하려는 작업과 관련이 있지 않다.</span> 그렇기 때문에 위 코드를 구성함으로써 현재 실행에 관련이 있는 프로젝트만 구성을 하게 된다.

### 2️⃣ `kotlin-precompiled-script-plugin` 을 별도로 적용해줘야 할까?

- 해당 플러그인은 `kotlin-dsl`에 내장되어 있기 때문에 별도 선언이 필요 없다.

### 3️⃣ 필요한 스크립트 파일 선언

`xxx.gradle.kts` 형식으로 스크립트 파일을 선언하게 되는데, 이때 xxx는 실제로 플러그인을 사용할 때 쓰는 id가 된다.

```kotlin
// droidknights: build-logic/src/main/kotlin/droidknights.android.feature.gradle.kts

import com.droidknights.app.configureHiltAndroid
import com.droidknights.app.configureRoborazzi
import com.droidknights.app.libs

plugins {
    id("droidknights.android.library")
    id("droidknights.android.compose")
}

android {
    packaging {
        resources {
            excludes.add("META-INF/**")
        }
    }
    defaultConfig {
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }
}

configureHiltAndroid()
configureRoborazzi()

dependencies {
    implementation(project(":core:model"))
    implementation(project(":core:data"))
    implementation(project(":core:designsystem"))
    implementation(project(":core:domain"))
    implementation(project(":core:navigation"))
    implementation(project(":core:ui"))

    testImplementation(project(":core:testing"))

    val libs = project.extensions.libs
    implementation(libs.findLibrary("hilt.navigation.compose").get())
    implementation(libs.findLibrary("androidx.compose.navigation").get())
    androidTestImplementation(libs.findLibrary("androidx.compose.navigation.test").get())

    implementation(libs.findLibrary("androidx.lifecycle.viewModelCompose").get())
    implementation(libs.findLibrary("androidx.lifecycle.runtimeCompose").get())
}

```

- 마찬가지로 `extensions`를 활용해야 함
- Project에 종속이 되지 않기 때문에 추가적인 자동완성도 지원되지 않음

### 4️⃣ [추가] `android{}` 을 사용하려고 한다?

android{}를 plugin에서 사용할 수 있는 방법은 크게 두 가지이다.

### 1. 안드로이드 플러그인 사용을 직접 선언해준다.
    
그냥 사용하면 안드로이드 플러그인을 찾을 수 없다는 오류가 발생한다. 따라서 다음과 같이 안드로이드 플러그인 사용 선언을 해준다.

```kotlin
plubins {
    id("com.android.library")
}

android {
    buildFeatures.compose = true
    composeOptions {
        kotlinCompilerExtensionVerison = ..
    }
}

dependencies {
    // 위에서 추가한 안드로이드 플러그인 덕분에 implementation도 손쉽게 사용 가능
    implementation(platform(libs.androidx.compose.bom))
    //...
}
```
    
### 2. kotlin dsl을 활용해 extension을 정의해 사용한다.

- 실제 내부 코드를 이용해 직접 접근할 수 있게 된다.
    - kotlin dsl에서 제공하는 `android {}` 의 내부 코드를 보면 아래 내용과 비슷한 것이 작성되어 있는 것을 확인할 수 있음
- 빌드 시점에 검사하도록 함

```kotlin
internal val Project.androidExtension: CommonExtension<*, *, *, *, *, *>
    get() = runCatching { libraryExtension }
        .recoverCatching { applicationExtension }
        .onFailure { println("Could not find Library or Application extension from this project") }
        .getOrThrow()
```

```kotlin
androidExtension.apply {
    // 활용
}
```
    
### 5️⃣ 플러그인 사용하기

```kotlin
// buildSrc/src/main/java/convention.android.library.gradle.kts

// build.gradle.kts
plugins {
	id("convention.android.library")
}
```

<br>

## 🔗 [추가 내용] Gradle Type-safe project accessors

settings.gradle.kts에 다음과 같은 코드가 들어가는 것을 종종 확인할 수 있었다.

```kotlin
enableFeaturePreview("TYPESAFE_PROJECT_ACCESSORS")
```

![typesafe project accessors](/assets/images/typesafe_project_accessors.png)

해당 기능은 `gradle 7.0` 부터 제공한다. 이는 multi-project를 적용할 떄 `project(":some:path")`와 같은 코드를 build gradle에 추가해줘야 하는데 이때 `1. type-safety`와 `2. code completion` 기능을 제공해준다.

이렇게 된다면 직접 모듈의 이름을 작성할 때 발생할 수 있는 실수를 미연에 방지할 수 있다.

<br>

## 🔗 [추가 내용] desugaring이 뭘까?

```kotlin
compileOptions {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
    isCoreLibraryDesugaringEnabled = true // desugaringEnabled 설정
}
```

droidKnights, nowinandroid 코드를 확인하면 다음과 같이 desugaring을 적용한 것을 확인할 수 있었다. 여기서 `desugaring`은 syntax sugar가 적용된 코드를 컴파일 과정에서 낮은 API의 기기에서도 해석할 수 있게 해주는 것이다.

syntax sugar은 프로그래밍 언어의 문법을 더 쉽게 읽거나 표현할 수 있게 해주는 것을 의미한다. Java 8의 Stream Api 등이 syntax sugar에 해당한다.

따라서 API 24 이전 기기에서도 java 8+ API를 사용할 수 있도록 하는 방법이 바로 desugaring이다.

<br>

## 🔗 마무리하며

`gradle plugin`을 적용하고 보니, 각 모듈의 `build.gradle` 파일이 정말 깔끔해진 것을 확인할 수 있었다. 하지만 `gradle`, `kotlin-dsl`에 대한 기본적인 이해가 필요했기에 적용하는 과정에서 많은 이슈를 만나기도 했다.

build gradle의 중복 코드를 관리하는 것은 규모가 큰 프로젝트일수록 이점이 커질 것 같다. 몇 개의 모듈만 생성해도 반복되는 로직이 많아지는데 모듈의 개수가 100개를 넘어가는 큰 프로젝트에서는 위와 같은 방식을 적용해 빌드 로직을 한곳에서 관리하게 된다면 유지보수 과정에서 큰 편리함을 느낄 것 같다.

현재 `nowinandroid`, `droidknight` 프로젝트를 확인해본다면 `build-logic` 내부에 서로 다른 방식으로 빌드 로직들을 관리하고 있는 것을 확인할 수 있다. 필자도 해당 방식을 적용하며 두 레포의 코드를 많이 참고하였다. nowinandroid의 경우, `binary plugin`을 적용하고 있으며, droidknights는 `precompiled script plugin` 방식을 적용하고 있다.

처음 적용할 때는 해당 레포들을 보고 따라하기 보다는 [droidkights에 올라온 다음 영상](https://www.youtube.com/watch?v=7iag6zpGd98)을 참고해 `gradle / kotlin dsl`에 대한 어느 정도 이해를 갖추고 레포들을 참고해 gradle plugin을 구성해보는 것을 추천한다!

## 🔗 참고 자료
- [https://www.youtube.com/watch?v=7iag6zpGd98](https://www.youtube.com/watch?v=7iag6zpGd98)
- [https://www.slideshare.net/slideshow/gradle-kotlin/261178195#77](https://www.slideshare.net/slideshow/gradle-kotlin/261178195#77)
- [https://github.com/droidknights/DroidKnightsApp/blob/main/build-logic/src/main/kotlin/com/droidknights/app/KotlinAndroid.kt](https://github.com/droidknights/DroidKnightsApp/blob/main/build-logic/src/main/kotlin/com/droidknights/app/KotlinAndroid.kt)
- [https://github.com/android/nowinandroid](https://github.com/android/nowinandroid)