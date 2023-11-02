---
title: "[Multi-Module] navigation with multi-module"
excerpt: "멀티 모듈에서 네비게이션 사용하는 법"
categories:
  - Android
tag:
  - Android
  - multi-module
  - navigation

last_modified_at: 2023-11-02
toc: true
toc_sticky: true
search: true
---

## 👩🏻‍💻 멀티모듈에서 네비게이션 사용하기

현재 feature 모듈을 도메인별로 `feature:main`, `feature:login`으로 나눈 상태이다. 이렇게 모듈을 분리했을 때, 모듈 사이 엑티비티 전환은 어떻게 진행해야 하는지를 이번 게시글을 통해 다뤄보고자 한다.

## 👩🏻‍💻 멀티모듈 구조

우선 프로젝트에 멀티모듈을 적용했다면 크게 [공식문서에 나온 것 과 같이](https://developer.android.com/topic/modularization?hl=ko) 모듈이 분리될 것이다.

<img src = "https://drive.google.com/uc?id=1RP89UZCAnSh-2DZKkBsrqUexk6F87xc8">

여기서 우리가 짚고 넘어갈 부분이 있다.

- <span style = "background-color:#fff5b1">각 기능 모듈은 직접 또는 간접적으로 app 모듈에 포함된다.</span>
  ```kotlin
    implementation(project(":domain"))
    implementation(project(":data"))
    implementation(project(":common-ui"))
    implementation(project(":feature:main"))
    implementation(project(":feature:login"))
  ```

  아마 모듈을 생성할 때마다 다음과 같이 app 모듈에 dependency를 걸어주었을 것이다.

  ### - app 모듈의 역할

  app 모듈은 각 feature 모듈에 종속되어 있다는 점에서 우리는 `app 모듈`을 활용해 각 모듈 사이 navigation 처리를 진행할 수 있다.

## 👩🏻‍💻 진행 과정

### 1. app module이 모든 feature module에 의존성이 존재함
  - <span style = "background-color:#fff5b1">모둘 사이 실제 navigate 로직이 정의됨</span>

  <br>

### 2. feature 모듈에서는 app 모듈을 모르는데?
  - feature 모듈이 공통적으로 아는 <span style = "background-color:#fff5b1">common-ui</span> 모듈을 사용한다.
    - navigator Interface 정의
      ```kotlin
        interface FakeAttNavigator {
            fun navigateToMain(context: Context)
        }
      ```
    - navigator Injector 정의
      - 뒤에 코드 참조!

  <br>

### 3. app에서는 common-ui를 안다!
  - common-ui의 `navigator Interface`를 구현
    ```kotlin
      class AttNavigator @Inject constructor(): FakeAttNavigator {
          override fun navigateToMain(context: Context) {
              val intent = Intent(context, MainActivity::class.java)
              context.startActivity(intent)
          }
      }
    ```
  - hilt 모듈을 생성해 interface와 구현체 bind
    ```kotlin
      @InstallIn(SingletonComponent::class)
      @Module
      interface NavigatorModule {
          @Binds
          fun bindAttNavigator(
              attNavigator: AttNavigator
          ): FakeAttNavigator
      }
    ```

<br>

### 4. common-ui에서 hilt를 통해 주입되는 실제 navigator 활용

  ```kotlin
    sealed interface AttInjector {

        @EntryPoint
        @InstallIn(ActivityComponent::class)
        interface AttNavigatorInjector {
            fun attNavigator(): FakeAttNavigator
        }

    }
  ```

<br>

### 5. feature 모듈에서 common-ui를 활용해 navigate 처리

```kotlin
  // attNavigator 가져오기
      private val attNavigator: FakeAttNavigator by lazy {
        EntryPointAccessors.fromActivity(
            requireActivity(),
            AttInjector.AttNavigatorInjector::class.java
        ).attNavigator()
      }

  // navigate 처리
    attNavigator.navigateToMain(requireContext())
```

<br>

## 👩🏻‍💻 추가적인 개념

### [EntryPoint](https://dagger.dev/hilt/entry-points.html)

- <b>EntryPoint가 뭐지?</b>

  > An entry point is the boundary where you can get Dagger-provided objects from code that cannot use Dagger to inject its dependencies. 

  - dagger가 제공하는 객체를 get할 수 있는 경계점이라 생각하면 될 것 같다.

  <br>

- <b>EntryPoint를 언제 사용해야 하는데?</b>

  > You will need an entry point when interfacing with non-Dagger libraries or Android components that are not yet supported in Hilt and need to get access to Dagger objects.

  - hilt에 의해서 아직 제공되지 않는 라이브러리나 android component의 interface를 얻고자 할 때 필요로 한다.

<br>

- <b>어떻게 사용하는데?</b>

  ```kotlin
    @EntryPoint
    @InstallIn(SingletonComponent::class)
    interface FooBarInterface {
      @Foo fun bar(): Bar
    }
  ```

  1. interface를 정의하고 제공하고자 하는 객체에 access할 수 있는 함수를 포함시킨다.
  2. `@EntryPoint`를 붙이고, `@InstallIn`을 통해 entryPoint를 어디에 설치할지를 정한다.

  ```kotlin
    attNavigator = EntryPoints.get(requireActivity(), AttNavigatorInjector::class.java).attNavigator()
  ```
  -> 공식문서에 나온 방법

  ```kotlin
    private val attNavigator: FakeAttNavigator by lazy {
          EntryPointAccessors.fromActivity(
              requireActivity(),
              AttNavigatorInjector::class.java
          ).attNavigator()
      }
  ```
  -> delegate 패턴으로 사용