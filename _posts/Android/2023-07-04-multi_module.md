---
title: "[Android] Multi-Module"
excerpt: "multi-module을 적용하기 위해 필요한 개념들"
categories:
  - Android
tag:
  - android
  - Multi Module
  - Version Catalog
  - BuildSrc

last_modified_at: 2023-07-04
toc: true
toc_sticky: true
search: true
---

지금까지 나는 프로젝트를 세팅하면 기본적으로 주어지는 app 모듈 하나로만 프로젝트를 구성해 개발해왔다.

그러다 최근 멀티모듈이라는 개념에 대해 알게 되었다. 이를 학습하고 적용하는 과정을 이번 게시글을 통해 쭉 정리해보겠다!

# 👩🏻‍💻 모듈화?

우선 모듈화란 **패키지 혹은 라이브러리 의존 사항들을 큰 단위로 분류**해둔 것을 의미한다.

그렇다면 그냥 app 모듈 하나만으로 개발을 하면 안되는 것일까? 우선, 가능은 하다! 하지만 그렇게 했을 때 문제점이 발생하게 된다.

app 모듈 하나에 모든 코드를 넣게 되면 누구나 해당 코드에 접근할 수 있어진다. 즉, <span style = "background-color:#fff5b1">모든 곳에서 해당 코드에 접근해 사용할 수 있어진다는 것이다.</span> 따라서 협업 시에는 <span style = "background-color:#fff5b1">모듈화를 통해 코드 접근을 제한하는 것</span>이 좋다!

<br>

# 👩🏻‍💻 모듈화 장점

그렇다면 모듈화를 통해 얻을 수 있는 장점에 대해 본격적으로 알아보자!

1. `reusability`
* 코드 공유
* <span style = "background-color:#fff5b1">**동일한 기반으로 여러 애플리케이션 구축 가능**</span>
  * 각 기능에 대한 각각의 모듈을 생성한다.

2. `customizable delivery`
* <span style = "background-color:#fff5b1">**맞춤형 배포**</span>

3. `ownership`
* <span style = "background-color:#fff5b1">**책임 강제화**</span>
* 각 모듈에는 코드 유지, 버그 수정, 테스트 추가 및 변경 사항 검토 담당 전담 소유자가 존재한다.

4. `encapsulation`
- 각 모듈은 다른 모듈에 대해 <span style = "background-color:#fff5b1">**최소한의 지식**</span>을 갖는다.
- isolated code는 읽고 이해하는 데 쉽다.

5. `build time`
- 다중 모듈 -> <span style = "background-color:#fff5b1">**빌드 성능 향상**</span>
  * 모듈들은 책임이 다르기 때문에 알아야 하는 라이브러리 숫자도 줄어들게 되기 때문이다.


<br>

# 👩🏻‍💻 low coupling & high cohesion

결국 모듈화를 통해 얻고자 하는 것은 <span style = "background-color:#fff5b1">낮은 결합도, 높은 응집력</span>이다.

1. low coupling: 낮은 결합력
* <span style="background-color:#fff5b1">한 모듈의 변경이 다른 모듈에 영향을 미치지 않거나 최소한의 영향을 미쳐야 한다.</span>
* 다른 모듈의 내부 동작에 대해 알 필요가 없다.
* 모듈이 최대한 서로 독립적이어야 한다.

2. high cohesion: 높은 응집력
* <span style="background-color:#fff5b1">모듈은 맡은 일이 명확히 규정되어 있고 특정 도메인 지식의 범위를 벗어나지 않도록 한다.</span>
* ex) 책과 결제 관련 코드를 동일한 모듈에 혼합하는 것은 적합할까?
  * NO!
  * 서로 다른 기능 영역(domain)
  * 서로 자주 상호작용하지 않는다.

<br>

# 👩🏻‍💻 모듈화 유형
![multi-module](/assets/images/multi_module.png)

위 그림은 <span style = "background-color:#fff5b1">모바일 클린 아키텍쳐에 기반한 모듈 구조</span>이다.

## 1. App Module
  * <span style = "background-color:#fff5b1">여러 기기 유형을 타겟팅하는 경우, 기기별로 앱 모듈을 정의하는 것이 좋다.</span>
    * 플랫폼별 종속 항목을 구분하는 데 도움이 된다.
  * 기능 모듈에 종속적

## 2. Feature Module
  * 화면 또는 밀접하게 관련된 일련의 화면에 해당하는 독립적인 앱의 기능(ex: 가입 흐름, 결제 흐름)
  * <span style = "background-color:#fff5b1">단일 feature가 단 하나의 view 또는 네비게이션 대상(하단바의)일 필요는 없다.</span>
  * 즉, 같은 도메인이라면 하나의 feature module로 구성하면 된다.

## 3. Data Module
* repository, data sources and model classes
* <span style = "background-color:#fff5b1">특정 도메인의 모든 데이터, 비즈니스 로직 캡슐화 -> 각 data module은 특정 도메인을 나타내는 데이터 처리</span>
* repository를 외부 api로 노출
* 외부로부터 모든 세부정보 및 데이터 소스 숨기기
  * <span style = "background-color:#fff5b1">같은 module의 repository를 통해 데이터 소스에 접근이 가능하다. 즉, 외부에 공개되지 않는다.</span>
  * kotlin의 `private`, `internal` 키워드를 통해 데이터를 숨길 수 있다.

## 4. Domain Module
* <span style = "background-color:#fff5b1">클린아키텍처에서 핵심은 domain이 가지는 의존성을 줄이는 것이다.</span>
  * 이렇게 했을 때 <span style = "background-color:#fff5b1">domain을 중심으로 서비스를 설계</span>할 수 있어진다.
* <span style = "background-color:#fff5b1">프레임워크에 종속성을 갖지 않아야 한다.</span>
  * 안드로이드 앱의 경우, 안드로이드 의존성을 갖지 않고 java와 kotlin을 활용해서만 코드를 작성해야 한다.

## 5. Common/Core Module
* 다른 모듈에서 공통으로 사용하는 코드가 포함된다.
* UI module
  * custom ui element
  * widget collection을 모듈에 캡슐화해 <span style = "background-color:#fff5b1">다른 feature에서 재사용이 가능해진다.</span>
* analytics module
  * 애널리틱스 추적기를 서로 관련 없는 components에서 사용할 경우가 많은데 그때 활용된다.
* network module
  * custom configuration이 필요할 때 유용하다.
  * dedicated to providing a http client
* utility module
  * 앱 전체에서 재사용되는 작은 코드

<br>

# 👩🏻‍💻 버전 관리

멀티 모듈에 대해 학습하고 있는데 갑자기 버전관리? 멀티모듈을 적용하면 <span style = "background-color:#fff5b1">각 모듈마다 gradle file을 가지고 dependency와 version을 정의하기 때문에 관리가 어렵다는 문제가 있다.</span> 따라서 BuildSrc나 Version Catalog를 적용해 <span style = "background-color:#fff5b1">프로젝트에서 사용되는 모든 dependency와 Version을 한 곳으로 모아 쉽게 관리할 수 있도록 하는 것이다.</span>

## BuildSrc

버전을 관리하는 방법 중 하나는 BuildSrc를 활용하는 것이다. 해당 방식을 활용하면 간단히 버전관리를 할 수 있지만 다음의 문제가 있다.

* 라이브러리 업데이트 정보를 알 수 없다.
* <span style = "backgroud-color:#fff5b1">약간의 수정만 있어도 재빌드를 한다.</span> -> 큰 단점


## Version Catalog

BuildSrc가 가지는 문제점을 해결하는 것이 바로 Version Catalog이다. 그래서 최근에는 version catalog를 활용해 많이들 버전 관리를 진행하는 것 같다. 이 또한, 적용하는 방식이 무척 간단하다.

* 기본적으로 toml 파일로 작성이 된다.
* [versions], [libraries], [plugins], [bundles] section을 나누어 정의할 수 있다.
* <span style = "background-color:#fff5b1">BuildSrc와 달리 renovate를 지원해줘 버전 관리가 쉬워진다.</span>
* <span style = "background-color:#fff5b1">버전의 영향을 받는 모듈만 rebuild가 된다.</span>

그렇다면 적용 예시 코드를 보자.

```kotlin
// libs.versions.toml 파일

#compose
compose-bom = "2023.08.00"
compose-appcompat-theme = "0.28.0"
compose-constraintlayout = "1.0.1"

# Other
jsoup = "1.13.1"


[libraries]
agp = { module = "com.android.tools.build:gradle", version.ref = "gradleplugin" }
kotlin = { group = "org.jetbrains.kotlin", name = "kotlin-stdlib", version.ref = "kotlin" }
...

[bundles]

androidx-ui-foundation = ["core-ktx", "appcompat", "lifecycle-viewmodel", "lifecycle-viewmodel-compose", "lifecycle-runtime-compose", "runtime-livedata", "constraintlayout", "activity", "fragment"]
okhttp = ["okhttp", "okhttp-logging-interceptor", "okhttp-bom"]
...

[plugins]

android-application = { id = "com.android.application", version.ref = "gradleplugin" }
android-library = { id = "com.android.library", version.ref = "gradleplugin" }
```

이를 gradle에서 다음과 같이 가져와 사용할 수 있다.

```kotlin
dependencies {
    implementation(project(":common-ui"))

    ...
    implementation(libs.appcompat)
    implementation(libs.material)
    implementation(libs.bundles.navigation)
}
```

<br>

# 👩🏻‍💻 Common gradle

멀티 모듈을 세팅할 때 학습할 부분 중 마지막 부분이다! 바로 `Common gradle`! 위에서도 언급했듯이 멀티모듈을 세팅하면 각 모듈마다 gradle 파일을 구성하게 된다. 이때 <span style = "background-color:#fff5b1">gradle 파일에서 공통적인 부분을 매번 작성되게 되는데 이러한 보일러 플레이트 코드를 없앨 수 있는 것이 바로 Common gradle이다. </span>

다음은 `common.gradle` 파일의 일부분이다.

```kotlin
if (hasLibraryPlugin || hasApplicationPlugin) {
    android {
        compileSdk = libs.versions.compileSdk.get().toInteger()
        defaultConfig {
            minSdk = libs.versions.minSdk.get().toInteger()

            if (hasLibraryPlugin) consumerProguardFiles("consumer-rules.pro")
            testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        }
        ...
    }
    ...
}
```

이렇게 구성한 `common.gradle`을 어떻게 적용할 수 있을까?

다음과 같이 필요한 gradle 파일에 하나하나 적용할 수도 있다.

```kotlin
// 각 모듈에 해당하는 gradle 파일
plugins {
    id 'com.android.application'
}

apply from: '../gradle/common.gradle'
```

하지만 모든 gradle에 한꺼번에 적용도 가능하다! 이때는 `project` 레벨의 `gradle` 파일을 활용한다.

```kotlin
subprojects {
    afterEvaluate {
        project.apply("$rootDir/gradle/common.gradle")
    }
}
```
- `subprojects`: 해당 프로젝트에 포함된 모든 모듈의 build.gradle을 제어
- `afterEvaluate`: 빌드 스크립트의 모든 구성 요소가 평가되고 설정된 후 실행

<br>

# 😌 정리

![multi-module-result](/assets/images/multi_module_foldering_result.png)

![multi-module-foldering-result](/assets/images/multi_module_result.png)

지금까지 멀티 모듈을 세팅하기 위해 필요한 필수적인 개념 + 부가적인 개념을 하나의 게시글로 쭉 정리해봤다. 이를 통해 왜 멀티모듈이 필요한지? 멀티모듈을 구성하기 위해서는 어떠한 개념들이 필요한지를 꼬리에 꼬리를 물며 이해할 수 있었다. 

개념을 정리하며 적용법도 함께 정리해볼까했는데 게시글 양이 어마어마하게 길어질 것 같아 개념만을 모아 정리해보았다! 적용방법은 안드로이드 공식 문서나 잘 정리된 블로그를 참고한다면 쉽게 적용할 수 있을 것이다!

`지식 + 1`된 기분을 얻고 이렇게 이번 게시글을 마무리하도록 한다.

마지막으로 적용방법을 러프하게 정리한 [노션 페이지](https://separated-stick-863.notion.site/Modularization-c27b49e0bc1d4dfc9d0b7347cad5dda0?pvs=4)를 함께 첨부한다!

<br>

# 📃참고
* SWM 모바일 특강
* udemy 강의: complete multi-modular architecture for android development
* <https://two22.tistory.com/81>
* <https://developer.android.com/studio/build/agp-upgrade-assistant?hl=ko>
* <https://medium.com/@callmeryan/gradle-version-catalog-728111fa210f>