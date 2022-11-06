---
title: "[Android] Databinding"
excerpt: "databinding의 사용법"
categories:
  - Android
tag:
  - databinding

last_modified_at: 2022-11-04
toc: true
toc_sticky: true
search: true
---

# 🙋‍♀️ Databinding?

> Databinding library(**Android Jetpack** 구성요소)?
  : 선언적 형식으로 레이아웃의 **UI 구성요소**를 **앱의 데이터 소스**와 **결합**할 수 있는 지원 라이브러리이다.

* databinding은 **MVVM** 패턴 구현 시, **view와 viewmodel 사이의 의존성을 낮추는 역할**로 거의 필수적으로 사용된다.

<br>

그렇다면 이제부터 Databinding의 사용법에 대해서 자세히 살펴보자!! 필자는 [Android DataBinding Codelab](https://developer.android.com/codelabs/android-databinding#4)을 통해 학습한 내용을 정리해보고자 한다.


<br>

# 🙋‍♀️ 사용법

## ⌨️ build.gradle에 dataBinding 추가!

우선 DataBinding을 사용하기 위해 gradle 파일에 다음의 코드를 추가해야 한다!

```
android {
...
    dataBinding {
       enabled true
    }
}
```

## ⌨️ layout 수정하기 

layout을 수정하는 법은 크게 다음과 같다.

* 기존 layout을 data binding layout으로 변경하는 법
  * 기존 레이아웃 **전체를 `<layout>` 태그로 감싼다.**
  * **layout variables**가 필요하면 추가한다.
  * layout variables을 사용해 data binding을 진행한다.

<br>

### 1. `<layout>`으로 감싸기

<img src = "https://drive.google.com/uc?id=11qr67WRVzoWv6igs47DOgTvWC9-bU6t_" width = 600 height = 500>

* 위 사진처럼 **자동으로 data binding layout으로 변경** 가능하다!
  * 레이아웃에서 우 클릭 -> show context actions -> **convert to data binding layout** 선택!

<img src = "https://drive.google.com/uc?id=1ZxSIsPnfidWsOHF8OfwTJO77njX5vCli" width = 600 height = 500>

수정을 완료하면 다음과 같은 형태일 것이다!

<br>

### 2. 필요한 layout variable 정의하기

```
    <data>
        <variable name="name" type="String"/>
        <variable name="lastName" type="String"/>
    </data>
```
* 위에서는 String 타입의 layout variable을 두개를 선언했다.

```
    <TextView
            android:id="@+id/plain_name"
            android:text="@{name}" 
    ... />
```
* layout expression 사용해 정의한 변수를 사용하기
  * `@`으로 시작하며 `{}`를 통해서 값을 감싼다.


## ⌨️ activity 수정: inflation 변경, ui call 제거

이렇게 한다면 layout은 수정이 완료되었다. 그렇다면 이제 activity를 수정해줄 차례이다!

dataBinding을 사용하면 inflation은 다른 방식으로 진행된다. 아래의 과정을 따라가보자!

```kotlin
  setContentView(R.layout.plain_activity)
```
위의 코드는 아래의 코드를 통해 대체가 된다.

```kotlin
  val binding : PlainActivityBinding =
    DataBindingUtil.setContentView(this, R.layout.plain_activity)
```
Binding class는 라이브러리에 의해 자동적으로 생성된다. 위의 경우, `PlainAcitivyBinidng`의 Binding class가 생성되게 된다.

```kotlin
  binding.name = "Your name"
  binding.lastName = "Your last name"
```
layout varialbe 값을 설정하면 마무리가 된다! 이제 결과를 확인해보면 설정한 값이 화면에 출력되는 것을 볼 수 있다.


## ⌨️ DataBinding: Handler User Event

DataBinding을 사용해서 **event handler**도 처리할 수 있다. 그렇다면 DataBinding에서 처리하지 않는 이전 코드를 먼저 확인한 후, 해당 코드의 문제점에 대해 알아보자!

```kotlin
  android:onClick="onLike"
```

```kotlin
    fun onLike(view: View) {
        viewModel.onLike()
        updateLikes()
    }
```
위는 버튼에 설정된 onClick 속성이다. 여기서 `onLike`는 **activity에 정의**된 클릭 이벤트이다. 그렇다면 이렇게 설정할 경우 문제가 무엇일까?

만약 해당 메소드가 정확히 정의되어 있지 않으면 런타임에서 오류가 발생할 것이다. 하지만 이를 **Databinding**을 통해 설정한다면 **컴파일 타임에 확인이 되기 때문에** 위 방법보다 안전하다.

```kotlin
  android:onClick="@{() -> viewmodel.onLike()}"
```

아마 위 코드를 보다가 'viewmodel이 뭐지?'라는 의문을 가질 수도 있다. 

DataBinding은 위에서 언급했듯이 거의 **ViewModel과 함께** 쓰일 때가 많다. 위 코드는 그 예시 중 일부분이므로 코드 자체의 의미보다는 **DataBinding을 통해 Handler도 처리할 수 있다는 사실에 집중하길 바란다!**

<br>

## ⌨️ Binding Adadpter를 통해 custom attribute 만들기

요소에 적용되는 **속성을 우리가 직접 만들 수 있다.** 이때 사용하는 파일이 바로 util에 있는 `BindingAdapters.kt`이다.

🙄 여기서 상황을 가정해보자. pressbar의 값이 0일 때는 화면에 보여지지 않고, 0을 넘을 경우 화면에 표시되게 만드는 작업을 진행한다고 가정하자.

우리는 `BindingAdapter.kt` 파일의 다음의 함수를 추가해 속성을 생성할 수 있다.

```kotlin
    @BindingAdapter("app:hideIfZero")
    fun hideIfZero(view: View, number: Int) {
        view.visibility = if (number == 0) View.GONE else View.VISIBLE
    }
```
* `("app:hideIfZero")`: xml에서 사용할 속성명을 정의한다.
  * `("hideIfZero")`만 작성해도 된다.

* BindingAdapter는 기본적으로 파라미터 두개를 갖는다.
  * 첫 번째는 해당 method를 속성으로 사용할 view이므로 해당 view에 맞는 타입으로 작성해야 한다.
  * 두 번째는 생성한 속성으로 들어오는 값이다.

```kotlin
    <ProgressBar
            android:id="@+id/progressBar"
            app:hideIfZero="@{viewmodel.likes}"
...
```
이렇게 사용했을 때 위 함수에 들어가는 **두 번째 parameter 값**은 **viewmodel.likes**가 된다.

<br>

## ⌨️ 여러 parameter를 가지는 Binding Adapter 생성하기

사용법만 조금 달라졌을 뿐 기본적인 방식은 똑같다.

```kotlin
@BindingAdapter(value = ["app:progressScaled", "android:max"], requireAll = true)
fun setProgress(progressBar: ProgressBar, likes: Int, max: Int) {
    progressBar.progress = (likes * max / 5).coerceAtMost(max)
}
```
* 여기서 `requireAll`은 말그대로 모든 파라미터가 필요한지의 여부를 나타낸다.
  * **ture**: 모든 파라미터가 xml에 정의되어 있어야 한다.
  * **false**: 없는 파라미터의 경우, **null**이거나 **flase**, **0(boolean의 경우)**이어야 한다.
* 여러 개의 파라미터를 `@BindingAdapter(("progressScaled", "max"))`와 같이 정의할 수 있다.


```kotlin
        <ProgressBar
                android:id="@+id/progressBar"
                app:hideIfZero="@{viewmodel.likes}"
                app:progressScaled="@{viewmodel.likes}"
                android:max="@{100}"
```

위와 같이 필요한 파라미터를 모두 정의하게 되면 위에서 생성한 binding adaptor를 사용할 수 있게 된다.

<br>

## ⌨️ Binding Adaptor를 view(activity, fragment)에서 사용하는 법

아래와 같이 Binding Adaptor의 함수명을 사용해 view에서 사용 가능하다!

```kotlin
  setProgress(binding.progressBar, viewModel.likes, 100)
```

<br>

  🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃참고
* <https://brunch.co.kr/@mystoryg/175>
* <https://developer.android.com/codelabs/android-databinding#6>
* <https://velog.io/@jojo_devstory/Android-Databinding%EC%9D%84-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90>
* <https://ppost.tistory.com/entry/Databinding-2-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B0%94%EC%9D%B8%EB%94%A9-BindingAdapter-%EC%82%AC%EC%9A%A9%EB%B2%95>