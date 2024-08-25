---
title: "[Annotation] Custom Annotation 활용 여정"
excerpt: "lifecycle callback with custom annotation!"
categories:
  - Android
tag:
  - Android
  - annotation
  - AnnotationProcessor

last_modified_at: 2024-08-25
toc: true
toc_sticky: true
search: true
---


## [Annotation?](https://kotlinlang.org/docs/annotations.html)

- 자바 소스 코드에 추가할 수 있는 메타 데이터의 한 형태
    - annotation은 소스 파일에서 읽을 수도 있고
    - 컴파일러에 의해 생성된 클래스 파일에 내장되어 읽힐 수 도 있고
    - runtime에 JVM에 의해 유지되어 리플렉션에 의해 읽힐 수 도 있다.
1. 특정 코드에 달아 어떤 의미를 부여하거나 기능을 주입할 수 있다.
2. [annottation으로 코드 검사를 개선할 수 있다.](https://developer.android.com/studio/write/annotations?hl=ko)
    
    ex) `@StringRes`, `@IntRange`, `@RequirePermission`...
    

## Annotation의 세 가지 종류

- kotlin/android에 내장되어 있는 `built in annotation`
- annotation에 대한 정보를 나타내기 위한 `meta annotation`
- 개발자가 직접 만드는 `custom annotation`
    - using reflection
    - using code generation(with annotation processor)

## 각 종류에 대한 설명

### 1) built in annotation

평소 우리가 사용하는 annotation이 여기에 해당

ex) `@StringRes`, `@IntRange`, `JvmOverloads`...

### 2) meta annotation

우선 annotation을 선언하게 된다면 다음과 같이 선언한다.

```kotlin
annotation class LifecycleLogger 
```

여기에 `meta-annotation`을 달아 annotation 사용에 제한을 걸 수 있다.

### 1️⃣ `@Target`

- 해당 annotation을 어디에서 사용 가능한지 제한
- class, functions, properties, expressions..

### 2️⃣ `@Retention`

- annotation의 scope를 제한
- 종류
    
    > source
    > 
    > - comile time에만 유용 / 빌드된 binary에는 포함되지 않음
    > - ex) `@suppress` 는 개발 중 warnning이 뜨는 것을 무시하도록 하는 annotation, 이와 같이 개발 중에만 유용, binary에는 포함될 필요 없음
    
    > binary
    > 
    > - compile time과 binary에도 포함되지만 `reflection을 통해 접근할 수 없음
    
    > runtime
    > 
    > - compile time과 binary에 포함됨
    > - reflection을 통해 접근 가능
    > - default로 사용됨

### 3️⃣ `@Repeatable`

- 한 요소에 annotation이 중복으로 사용될 수 있는지

### 4️⃣ `@MustBeDocumented`

- generated documentation에 해당 annotation도 포함될 수 있는지
    - 주로 library 만들 때 사용

```kotlin
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
@Repeatable
@MustBeDocumented
annotation class LifecycleLogger 
```

## Annotation의 장점

그렇다면 우리가 평상 시에 자주 사용하는 annotation의 장점은 과연 무엇일까?

1. 빠르다.
    - annotation processor는 javac 컴파이럴의 일부로, 모든 처리가 `컴파일 시간`에 발생한다.
2. 일반적인 annotation의 경우 리플렉션이 사용되지 않는다.
    - DI 라이브러리, JSON 직렬화/역직렬화 라이브러리 등 런타임 annotation으리 경우 리플렉션이 내부적으로 사용된다.
3. boilerplate code를 생성해준다.

## Custom Annotation 만들기: Refelction

- [reflection 방법](https://blog.gangnamunni.com/post/kotlin-annotation/)
    - 런타임에 코드 구조를 변경
    - 어플리케이션의 성능 저하를 초래함
        - reflection한 함수의 경우, 파라미터의 개수가 맞는지, 파라미터의 타입이 정확한지 확인하는 작업을 진행
        - `JIT compiler가 한 번만 할 작업을 런타임에 매번 진행`

## [Custom Annotation 만들기: Annotatiaon Processor를 통한 code generation](https://charlezz.com/?p=1167)

### Annotation Processor?

우리가 프로젝트를 만들 때 거의 필수적으로 적용하는 plugin인 `kapt`, `ksp` 가 바로 Annotation processor 중 한 종류이다. 즉, 단어를 통해서도 알 수 있듯이 annotation을 처리하는 역할을 하는 것이다.

### Annotation Processor의 동작 방식

1. 자바 컴파일러가 컴파일을 수행한다.(이때, 자바 컴파일러는 annotation processor를 미리 알고 있어야 한다.)
2. 실행되지 않은 annotation processor들을 수행한다.
    - 각 processor는 모두 각자 역할에 맞는 구현이 존재한다.
3. processor 내부에서 annotation이 달린 element(변수, 메소드, 클래스 등)을 처리한다.
    - 이때 보통 boilerplate code가 생성된다.
4. 컴파이러가 모든 annotation processor가 실행되는지 확인하고, 그렇지 않다면 위 작업을 반복한다.

<img width="664" alt="스크린샷 2024-08-25 오후 5 10 36" src="https://gist.github.com/user-attachments/assets/d47dd68d-995f-4ea2-94ab-acff31e9e181">

출처: https://charlezz.com/?p=1167    

## Reflection을 활용해 Annotation Processor 정의

나의 경우 처음 `AbstractProcessor`를 활용해 custom annotation을 처리하고 하였다. 하지만 나의 상황의 경우 `AbstractProcessor`를 사용하는데 다음의 문제와 고민이 생기게 되었다.

> 🫥 필자의 경우는 annotation을 통해 처리하려는 로직이 android와 관련된 로직이었다.
> 
> 
> > `처음 접근`
> > 
> 
> Android 모듈(a) 내 자바 라이브러리를 추가하고, `AbstractAnnotation을 상속한 AnnotationProc`를 정의한 뒤, 실제 프로젝트(또 다른 Android 모듈 b)에 AnnotationProcessor를 등록해주면 되겠다!
> 
> > `왜 해당 접근이 좋지 않을까?`
> > 
> 
> ### `AnnotationProcessor의 목적을 다시 한 번 생각해보자`
> 
> 1. annotation processor는 `compile time`에 실행되고 마무리된다.
> - java, kotlin에서 컴파일 타임에 동작한다.
> - 즉, 런타임에 동작하는 코드에는 직접적으로 영향을 주지 않는다.
> 
> ⇒ `lifecycle callback`은 런타임에 동작하는 콜백들이다. 만약 컴파일 타임에 실행되는 AnnotationProcessor 로직과 런타임 로직이 함께 존재한다면 두 가지 로직이 명확히 구분되지 않게 된다.
> 
> > `적용할 수 있는 다른 방식은 뭘까?`
> > 
> 
> 결국 java 라이브러리에서 제공하는 `AbstractProcessor`를 통해서 AnnotationProcessor를 정의하기 보다는 해당 로직들을 custom해서 작성하자!
> 

### 1. annotation 모듈 생성

> 대부분의 customAnnotation 관련 예제들을 본다면 `annotation`, `annotation-processor` 두 가지 모듈을 `java/kotlin library`로 생성하는 것을 확인할 수 있다.
> 
> 
> > 🫥  왜 anootation, annotation processor 각각에 대한 모듈을 생성하는 걸까?
> > 
> > - 필자의 생각에는 컴파일러가 annotation processor에 대한 정보를 알고 있어야 한다고 위에서 말했던 것처럼 annotation processor를 build 파일에서 등록해주어야 하기 때문이라 생각한다.

> 🫥 굳이 모듈까지 생성해야 하는 걸까?
> 
> - 모듈화를 통한 코드의 재사용성
> - 명확한 책임 분리 - annotation만을 다룸
- `annotation` 모듈을 생성하고 프로젝트 내부에서  `custom AnnotationProcessor` 로직을 정의하자.

### 2. custom annotation processor 로직 작성 - Activity

```kotlin
object ActivityLifecycleLogProcessor : ActivityLifecycleCallbacks {
    private const val TAG = "ActivityLifecycle"

    override fun onActivityCreated(p0: Activity, p1: Bundle?) {
        if (checkContainAnnotation(p0)) {
            registerFragmentLifecycleCallbacks(p0)
            Log.i(TAG, p0.javaClass.simpleName + ":" + "onActivityCreated")
        }
    }

    override fun onActivityStarted(p0: Activity) {
        if (checkContainAnnotation(p0)) {
            Log.i(TAG, p0.javaClass.simpleName + ":" + "onActivityStarted")
        }
    }

    override fun onActivityResumed(p0: Activity) {
        if (checkContainAnnotation(p0)) {
            Log.i(TAG, p0.javaClass.simpleName + ":" + "onActivityResumed")
        }
    }

    override fun onActivityPaused(p0: Activity) {
        if (checkContainAnnotation(p0)) {
            Log.i(TAG, p0.javaClass.simpleName + ":" + "onActivityPaused")
        }
    }

    override fun onActivityStopped(p0: Activity) {
        if (checkContainAnnotation(p0)) {
            Log.i(TAG, p0.javaClass.simpleName + ":" + "onActivityStopped")
        }
    }

    override fun onActivitySaveInstanceState(p0: Activity, p1: Bundle) {
        if (checkContainAnnotation(p0)) {
            Log.i(TAG, p0.javaClass.simpleName + ":" + "onActivitySaveInstanceState")
        }
    }

    override fun onActivityDestroyed(p0: Activity) {
        if (checkContainAnnotation(p0)) {
            unregisterFragmentLifecycleCallbacks(p0)
            Log.i(TAG, p0.javaClass.simpleName + ":" + "onActivityDestroyed")
        }
    }

    private fun checkContainAnnotation(targetActivity: Activity) =
        targetActivity.javaClass.isAnnotationPresent(LifecycleLog::class.java)

    private fun registerFragmentLifecycleCallbacks(targetActivity: Activity) {
        if (targetActivity !is AppCompatActivity) return

        targetActivity.supportFragmentManager.registerFragmentLifecycleCallbacks(
            FragmentLifecycleLogProcessor, true
        )
    }

    private fun unregisterFragmentLifecycleCallbacks(targetActivity: Activity) {
        if (targetActivity !is AppCompatActivity) return

        targetActivity.supportFragmentManager.unregisterFragmentLifecycleCallbacks(
            FragmentLifecycleLogProcessor
        )
    }
}
```

여기까지만 진행한다면 콜백이 트리거 되었을 때 어떻게 동작할지는 정의했지만, `이 콜백이 트리거되지는 않는다.` 따라서 우리는 Applicaation에 `registerActivityLifecycleCallbacks` 으로 해당 콜백 처리 로직을 등록해주어야 한다.

> 🫥 activity destroy될 때 등록한 fragmentLifcycleCallback 제거가 필요한 이유?
> 
> - activity가 destory 되어도 등록된 fragment lifecycle callback이 여전히 호출될 수 있다.
> - 이로 인해 activity와 fragment 사이 참조가 끊어지지 않아 메모리 누수가 발생할 수 있으므로 등록된 callback을 제거해 주어야 한다.

> 🫥 `supportFragmentManager` vs `fragmentManager`
> 
> - fragmentManager
>     - `android.app.fragment` 즉, 기본 Andorid Fragment와 관련된 기능을 제공한다.
>     - api 레벨 11 이상에서 사용할 수 있다.
> - supportFragmentManager
>     - `androidx.fragment.app.Fragment`를 사용하는 Androidx의 Fragment와 관련된 기능을 제공한다.
>     - api 호환성 제공
>         - 새로운 Fragment api와 호환성 문제를 해결하고, 이전 android 버전에서도 fragment를 사용할 수 있도록 한다.

### 3. lifecycleCallback 달기 - Application 파일 생성

```kotlin
class RaceGameApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        registerActivityLifecycleCallbacks(ActivityLifecycleLogProcessor)
    }
}
```

그리고 생성한 Application을 manifest 파일에 등록해줘야 한다.

```kotlin
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".app.RaceGameApplication" // applicaation 파일 등록
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
```

### 결과

<img width="401" alt="스크린샷 2024-08-25 오후 5 12 11" src="https://gist.github.com/user-attachments/assets/4866a5e1-da77-4d73-a96c-487e5dd02164">

## 참고

- https://blog.gangnamunni.com/post/kotlin-annotation/
- https://charlezz.com/?p=1167