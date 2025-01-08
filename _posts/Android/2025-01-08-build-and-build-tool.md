---
title: "[Build] build 그리고 build tool"
excerpt: "gradle / maven은 각각 어떤 작업을 진행할까?"
categories:
  - Android
tag:
  - Android
  - build
  - build tool
  - gradle
  - maven
  - dsl
  - kotlin dsl
  - groovy dsl

last_modified_at: 2025-01-08
toc: true
toc_sticky: true
search: true
---

## 🔗 들어가며

이전 게시글에서 `precompiled build script`를 통해 build plugin을 작성하는 방법에 대해 다뤘다. 하지만 당시 build에 대한 이해가 부족해 여러 부분에서 어려움을 겪었다.

안드로이드 프로젝트를 진행하면 필수적으로 `build.gradle` , `settings.gradle` 등 다양한 빌드 로직을 다루게 된다. 이번 게시글을 통해서 다시 한 번 빌드란 무엇인지를 정의하고, 안드로이드 개발에 주로 사용되는 `gradle`, `maven` 빌드 도구에 대해 다뤄보고자 한다.

## 🔗 빌드
`source code → 실행 가능한 소프트웨어 산출물`로 변환하는 과정을 의미한다. build 과정에는 소스 코드 컴파일, 필요한 파일 및 라이브러리 연결(링킹), 실행 파일 또는 배포 가능한 패키지(ex: applciation, library, jar 파일 등) 생성 등의 과정을 포함한다. 추가적으로 필요하다면 코드 최적화, 테스트, 문서 생성 등을 포함한다.

여기서 컴파일과의 차이점을 명확히 짚고 넘어가자면

- 컴파일: 단순히 <span style = "background-color:#fff5b1">source code → 기계어</span>로 변환하는 과정을 의미한다.
- 빌드: 컴파일을 포함하여, source code를 <span style = "background-color:#fff5b1">실행 가능한 소프트웨어 산출물로 만들기 위한 모든 과정</span>을 의미한다.

## 🔗 build tool

빌드 도구란 <span style = "background-color:#fff5b1">빌드 과정을 자동화하여 관리하는 기능을 하는 도구 (Build Automation Tool / 빌드 자동화 도구라고도 불림)</span>이다. 프로젝트 생성, 테스트 빌드, 배포 등의 작업을 진행한다.

이번 글에서는 대표적으로 현재 안드로이드 빌드를 진행할 때 활용되는 `Gradle`과 `Maven`에 대해 다룰 것이다.

## 🔗 [Gradle](https://docs.gradle.org/current/userguide/userguide.html)(안드로이드 공식 빌드 시스템)

안드로이드 프로젝트를 생성하면 빌드 관련 로직을 build.gradle 파일에 작성하고 있을 것이다. 이처럼 gradle은 <span style = "background-color:#fff5b1">안드로이드 공식 빌드 시스템으로 채택</span>이 되어 있으며, buildScript를 작성함으로써 빌드 로직을 구현할 수 있다. 

Gradle은 기본적으로 <span style = "background-color:#fff5b1">자바 환경 즉, JVM 위에서 실행</span>되기 때문에 Gradle을 사용하기 위해서는 `JDK / JBR`가 함께 설치되어 있어야 한다.(안드로이드 스튜디오를 설치하면 자동으로 JDK / JBR도 함께 설치가 된다.)

### 1️⃣ gradle build

<img width = 500 style="display: block; margin: 0 auto" src = "/assets/images/build_task_image.png"/>

안드로이드 프로젝트를 빌드를 하게 되면 다음과 같이 콘솔 창에 Task들에 대한 로그가 찍히는 것을 확인할 수 있다.

위와 같이 Gradle은 빌드 작업들을 하기 위해 `task`라는 개념을 사용한다.  모든 이용 가능한 task들은 gradle plugin이나 build scripts들로부터 나오게 된다.

프로젝트 속 `build scripts` 그리고 `플러그인`들은 이러한 task들에 대해서  `task dependency graph`를 구성하는데, 여기서 말하는 task dependncy graph란 무엇일까?

<br>

### 2️⃣ task Graph

<img width = 500 style="display: block; margin: 0 auto" src = "/assets/images/task_graph.png"/>

gradle은 task들을 실행하기 전에 위와 같이 task 사이의 관계를 나타내는 `task graph(= DAG, 방향 비순환 그래프)`를 먼저 생성한다. <span style = "background-color:#fff5b1">각 task들은 input과 output으로 서로 연결된다.</span>

그렇다면 이렇게 만들어진 task graph는 언제 어디에서 사용되는 것일까? 이를 확인하기 위해서 gradle의 빌드가 이루어지는 세단계를 살펴보자!

<br>

### 3️⃣ build lifecycle and phases

<img width = 500 style="display: block; margin: 0 auto" src = "/assets/images/build_phases.png"/>

gradle에서는 `build` 진행 시, 아래의 작업을 순서대로 진행한다.

1. <span style = "background-color:#fff5b1">Intialization</span>
    1. `settings.gradle(.kts)` files을 탐색한다
    2. `Settings` instance를 생성한다.
    3. settings를 통해 빌드를 구성할 projects를 결정한다.
    4. 각 project에 대해 `Project` instance를 생성한다.
2. <span style = "background-color:#fff5b1">Configurration</span>
    1. build에 참여하는 모든 프로젝트의 `build script(build.gradle(.kts)`를 평가한다.
    2. task 처리를 요청하기 위한 task graph를 만든다.
3. <span style = "background-color:#fff5b1">Execution</span>
    1. task schedule, execute 진행
    2. task 사이 dependency가 task 실행 순서를 결정
    3. task의 실행은 병렬적으로 진행될 수 있음

<br>

### 4️⃣ gradle 에서의 plugin?

`plugin`은 <span style = "background-color:#fff5b1">gradle build system을 확장하는 역할</span>을 한다. 이러한 plugin은 진행해야 하는 `task`들과 plugin 설정을 정의한다.

이러한 plugin은 build task 자동화, 외부 tool 및 서비스와의 통합, 빌드 과정끼리의 연결을 위해 필수적인 요소이다.

- plugin의 이점
    - <span style = "background-color:#fff5b1">재사용성 증가</span>
        프로젝트간 비슷한 빌드 로직의 중복을 제거한다.
    - <span style = "background-color:#fff5b1">modularity 향상</span>
        빌드 스크립트를 그룹화 하는 것이 가능하다.
    - <span style = "background-color:#fff5b1">로직 캡슐화</span>
        핵심적인 로직을 분리해서 관리할 수 있다.

<br>

### 5️⃣ gradle wrapper?

<img width = 500 style="display: block; margin: 0 auto" src = "/assets/images/gradle_wrapper.png"/>

안드로이드 프로젝트 gradle 폴더에 들어가면 `gradle-wrapper.jar` 가 존재하는 것을 확인할 수 있다.

> `JAR?(java archive file)`
> 
> 
> 여러 개의 자바 클래스 파일과 클래스들이 이용하는 관련 리소스 및 메타 데이터를 하나의 파일로 모아서 자바 플랫폼에 응용 소프트웨어나 라이브러리를 배포하기 위한 소프트웨어 패키지 파일 포맷
> 

그렇다면 `gradle-wrapper`의 역할은 무엇인지 간단히 다루고 넘어가자!

gradle 공식 문서를 확인한다면 gradle build를 실행하는데 권장되는 방식은 gradle wrapper를 사용하는 것이다. <span style = "background-color:#fff5b1">gradle wrapper는 선언된 gradle 버전을 확인해 미리 다운로드하는 스크립트</span>이다. 따라서 개발자들은 프로젝트 내에 wrapper를 사용하여 gradle project를 빠르게 실행할 수 있게 된다.

그렇다면 gradle wraper가 필요한 이유에 대해 몇 가지 정리하고 gradle에 대해서 마무리해보자.

1. <span style = "background-color:#fff5b1">빌드 환경의 일관성</span>
    
    협업 환경에서 gradle 버전이 상이하면 빌드 결과 또한 달라질 수 있다. 하지만 `gradle wrapper`를 사용함으로써 모든 개발자가 동일한 버전의 gradle을 사용하게 된다.
    
2. 빌드 과정 단순화
    
    프로젝트를 처음 접하는 개발자도 gradle wrapper를 통해 바로 빌드를 진행할 수 있다.
    
    > 이때 특정 개발자의 시스템 환경에 다른 버전의 gradle이 설치되어 있을 경우, wrapper에서는 필요한 gradle을 다운하여 로컬 파일 시스템에 저장하게 된다. (필요한 gradle 버전은 [`gradle-wrapper.properties`](http://gradle-wrapper.properties) 파일에 명시되어 있다)
    > 
    > 
    >  이를 통해 개발자마다 상이한 gradle 버전이 아닌 일관된 gradle 버전을 통해 빌드가 되게 된다.
    >

<br>

### 6️⃣ Gradle의 장점

아직 다루지는 않았지만 빌드 도구 중에서는 아래에서 다룰 `Maven`도 존재한다. 최근에는 Maven보다는 Gradle이 더 많이 쓰이고 있는 추세이다.

Gradle은 가장 최근에 나온 빌드 도구이다. 그리고 위에서 언급했듯이 안드로이드에서는 공식 빌드 시스템으로 gradle을 사용하고 있는데, 그렇다면 Gradle이 다른 빌드 도구들과 비교했을 때 제공하는 이점에 대해서도 조금 짚고 넘어가자.

1. 간결한 스크립트
    - `ant, mavem은 xml 문법 사용` → 태그 사용 등으로 복잡한 빌드 스크립트
2. 빌드 속도
    - <span style = "background-color:#fff5b1">gradle은 캐싱 및 병렬 빌드를 지원</span>하기 때문에 이전 빌드 도구보다 빌드 속도가 빠르다.
        
        (관련한 공식 문서를 참조로 두고 싶었지만, 무슨 이유에서인지 페이지 로딩이 되지 않아 해당 내용을 참조하고 있는 [블로그 글](https://medium.com/@yukeon97/spring-gradle-vs-maven-81b58ea3c96c)을 가져왔다.)

그렇다고 gradle만이 무조건 좋은 빌드 툴은 아니다. [다음 게시글](https://www.elancer.co.kr/blog/detail/270)을 확인해본다면 maven / gradle 각각의 비교가 자세히 나와있으며, 필요에 따라 알맞은 빌드 툴을 사용하는 것이 맞다.

<br>

## 🔗 Maven

안드로이드 개발을 하면 gradle 뿐 아니라 `Maven`도 함께 적용하고 있을 것이다.

둘이 함께 사용이 된다는 것은 서로 다른 역할을 하고 있다는 것인데…! 이번에는 Maven에 다뤄보자! 우선 Maven이란 무엇일까?

> 아파치 소프트웨어 재단에서 개발한 <span style = "background-color:#fff5b1">java 기반 프로젝트 빌드 및 관리 도구</span>
> 
> - <span style = "background-color:#fff5b1">라이브러리 관리 기능</span>도 내포하고 있다.
> - pom.xml 파일에 의존성을 정의할 경우, 필요한 의존성 및 참조하고 있는 의존성 모두 repository에서 검색해 자동으로 추가해준다.
> - lifecycle 개념이 도입되어, 해당 순서에 따라 빌드가 진행된다.

### 1️⃣ Maven에서의 Repository

```kotlin
repositories {
    google()
    mavenCentral()
    maven { url "https://jitpack.io" }
}
```

해당 부분에서는 이후에 나올 `repository` 라는 개념에 대해 잠깐 짚고 넘어가고자 한다. 다음과 같이 `settings` 파일에 repositories라는 block을 많이 사용해봤을 것이다. 그렇다면 여기서 말하는 `repository`가 무엇을 말하는 걸까?

maven에서는 <span style = "background-color:#fff5b1">산출물을 저장</span>하기 위해 repository를 사용한다. <span style = "background-color:#fff5b1">결국 maven을 통해 의존성을 다운로드 한다면 repository로부터 다운로드</span>가 되어지며, 그 산출물(installed, uploaded) 또한 이 repository에 저장되게 된다.

<br>

## 2️⃣ [안드로이드 외부 라이브러리 의존성 관리](https://docs.gradle.org/current/userguide/declaring_repositories.html)

안드로이드 프로젝트에서는 앱의 빌드는 Gradle 빌드 시스템과 AGP(android gradle plugin)를 주로 이루어진다. 그렇다면 `Maven`의 역할은 무엇일까? 바로 `외부 라이브러리 의존성 관리` 를 위해 maven이 활용된다. 

```kotlin
repositories {
    google() // public Google Android repository
    mavenCentral() // public Maven Central repository
    maven { url "https://jitpack.io" } // private / custom repository
}
```

위와 같이 `repositories` 블록을 활용해 다음과 같이 정의를 해준다면 <span style = "background-color:#fff5b1">gradle에서는 필요한 의존성을 어디서 다운할지를 알게 된다.</span> public / private / custom 등 다양한 repository를 선언하는 방식은 [다음 링크](https://docs.gradle.org/current/userguide/declaring_repositories.html)를 참고하자.

위 코드 중에서 `google()` 를 한 번 확인해보자. 이는 [google의 maven 저장소](https://maven.google.com/web/index.html)를 의미한다. 이를 통해서 Android sdk, androidx 라이브러리, google play services 등 google에서 제공하는 라이브러리들을 다운로드해 활용할 수 있게 된다!

<br>

## 🔗 DSL(domain specific language)

마지막으로 `DSL`에 대해 다루고 마무리하고자 한다. 안드로이드 스튜디오 giraffe부터 gradle의 default 언어가 `groovy dsl`에서 `kotlin dsl`로 바뀐 것을 알고 있을 것이다. 그렇다면 DSL이 무엇이며, kotlin dsl이 groovy dsl과 달리 어떤 추가적인 장점을 제공하는지 확인하고 넘어가자!

### 1️⃣ 도메인 특화 언어??

`domain specific language` 를 해석하면 도메인 특화 언어이다. 즉, <span style = "background-color:#fff5b1">특정 도메인에 특화되어 제작된 언어</span>라는 것이다.

우리가 주로 사용하는 프로그래밍 언어인 C, C++, java는 <span style = "background-color:#fff5b1">GPL(General Purpose Launguage)</span>이다. 즉, 범용 목적으로 사용되는 언어라는 것이다. DB 쿼리를 위한 SQL, 웹페이지를 위한 CSS, 정규 표현식, 빌드 자동화를 위한 Gradle DSL, Maven POM 등이 모두 DSL에 해당된다. 예시를 보면 DSL이 어떤 언어들을 의미하는지 감이 왔을 것이다.

이러한 DSL은 <span style = "background-color:#fff5b1">코드의 내부적인 로직을 숨기고 재사용성을 높여준다.</span> 따라서 사용하는 개발자 입장에서는 내부 로직을 몰라도 훨씬 간결하고 빠르게 코드를 작성할 수 있게 된다.

<br>

### 2️⃣ groovy dsl vs kotlin dsl

<img width = 500 style="display: block; margin: 0 auto" src = "/assets/images/kotlin_dsl.png"/>

안드로이드 스튜디오 Giraffe에서는 이제 공식 언어로 `kotlin DSL`이 지원된다. <span style = "baackground-color:#fff5b1">kotlin dsl은 코틀린 언어적인 특징을 활용한 gradle script 언어</span>이다. 그러다면 이전에 제공되던 groovy dsl과 어떤 차이점이 존재하는지 한번 확인해보자.

|  | kotlin DSL | groovy dsl |
| --- | --- | --- |
| 타입 시스템 | 정적 타입(빠르고 정확한 코드 힌트를 얻을 수 있으며, syntax error를 빠르게 감지할 수 있음) | 동적 타입 |
| 타입 안정성 | 높음 (컴파일 시 타입 검사) | 낮음 (런타임 시 타입 검사) |
| 코드 자동 완성 | 강력 | 약함 |
| 빌드 오류 발견 시점 | 컴파일 시점 (빠른 오류 발견) | 런타임 시점 (늦은 오류 발견) |
| 가독성 | 명확하고 간결함(kotlin 문법 활용) | 타입 정보 부재로 가독성이 떨어짐 |
| complication script 성능 | 느림 | 빠름 |
| complication 결과 저장 | gradle local / remove cache에 저장 → 즉 첫 빌드 이후 recomplication이 불필요해짐 | 제공하지 않음 |

## 🔗 마무리하며

이번 기회에 build에 대해 학습하면서 관련해서 알아야 할 개념이 정말 많다는 것을 깨달았다😇. gradle만 다루더라도 정말 많은 내용을 학습해야 할 것 같다는 생각이 들었다. 

build에 대해서 짚고 넘어가니 이전 시간에 진행했던 build-logic 개념이 조금 더 명확히 이해가 가는 것 같다. 커스텀 플러그인을 만듦으로써 반복해서 사용되는 로직을 재사용할 수 있었고, 이러한 플러그인을 통해서 gradle의 task들이 등록돼 빌드 시, 진행될 수 있구나를 이해할 수 있었다. 

아직 빌드에 대해 다루지 못한 내용이 많다. 빌드와 관련된 파일들이 각각 어떤 역할을 하는지에 대해서도 이해가 필요할 것 같으며, 크게 사용해보지 않았던 proguard에 대해서도 한번 학습하고 넘어가야 할 것 같다. 

빌드와 조금 더 친해지기를 바라며,,, 앞으로의 나 화이팅..!🤸

## 🔗 참고
- [https://docs.gradle.org/current/userguide/build_lifecycle.html](https://docs.gradle.org/current/userguide/build_lifecycle.html)
- [https://developer.android.com/build/gradle-build-overview](https://developer.android.com/build/gradle-build-overview)
- [https://docs.gradle.org/current/userguide/declaring_repositories.html](https://docs.gradle.org/current/userguide/declaring_repositories.html)
- [https://yozm.wishket.com/magazine/detail/1700/](https://yozm.wishket.com/magazine/detail/1700/)
- [https://android-developers.googleblog.com/2023/04/kotlin-dsl-is-now-default-for-new-gradle-builds.html](https://android-developers.googleblog.com/2023/04/kotlin-dsl-is-now-default-for-new-gradle-builds.html)
- [https://charlezz.com/?p=45140](https://charlezz.com/?p=45140)
- [https://www.dhiwise.com/post/kotlin-dsl-vs-groovy-which-should-you-use-for-build-files](https://www.dhiwise.com/post/kotlin-dsl-vs-groovy-which-should-you-use-for-build-files)