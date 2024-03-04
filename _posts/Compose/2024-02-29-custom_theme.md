---
title: "[Compose] custom theme 구성하기(1)"
excerpt: "compositionLocal에 대해서 먼저 알자!"
categories:
  - Compose
tag:
  - compositionLocal
  - custom theme
  - compositionLocalOf
  - staticCompositionLocalOf
  - compositionLocal deepdive

last_modifeid_at: 2024-02-29
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며
custom theme을 만들다 보면 `compositionLocal`을 생성하고 활용하는 코드를 만날 수 있다. 그렇다면 custom theme을 만들기 전에 우선 `compositionLocal`이 대체 무슨 역할을 해주는 것인지를 짚고 넘어가고자 한다.

<br>

# 🔗 CompositionLocal Deep Dive

## 🔗 CompositionLocal의 역할이 뭘까?

> Locally scoped data with CompositionLocal

compositionLocal은 <span style = "background-color:#fff5b1">로컬 범위 지정 데이터</span>를 제공할 수 있게 해준다. 그렇다면 로컬 범위 지정 데이터가 어떤 것을 의미할까?

아래 예시를 봐보자!

```kotlin
@Composable
fun MyApp() {
    // Theme information tends to be defined near the root of the application
    val colors = colors()
}

// Some composable deep in the hierarchy
@Composable
fun SomeTextLabel(labelText: String) {
    Text(
        text = labelText,
        color = colors.onPrimary // ← need to access colors here
    )
}
```

compose에서는 필요한 데이터를 `파라미터`를 통해 전달해줌으로써 데이터가 전반적으로 흐르게 된다. 그렇다면 앱 전반적으로 사용되는 color와 같은 sylte의 경우는 그럼 어떻게 해야 할까? 만약 파라미터로 전달해야 한다면 모든 컴포저블에 color와 같은 스타일 지정 파라미터가 필요할 것이다.

이때 사용하는 것이 <span style = "background-color:#fff5b1">compositionLocal</span>이다. 

> Compose offers `CompositionLocal` which allows you to create tree-scoped named objects that can be used as an implicit way to have data flow through the UI tree.

compositionLocal을 사용할 경우, <span style = "background-color:#fff5b1">트리 범위의 객체를 만들 수 있고 이를 해당하는 UI 트리 범위 내에서 사용할 수 있게 된다.</span>

만약 트리? 트리 범위?라는 말이 와닿지 않는다면 해당 공식 문서 [jectpack compose 단계](https://developer.android.com/jetpack/compose/phases?hl=ko)를 참고하자! 

## 🔗 CompositionLocal 사용 예: MaterialTheme 코드

그렇다면 사용 예시를 통해 조금 더 확실히 이해하고 넘어가자!

```kotlin
@Composable
fun MaterialTheme(
    colors: Colors = MaterialTheme.colors,
    typography: Typography = MaterialTheme.typography,
    shapes: Shapes = MaterialTheme.shapes,
    content: @Composable () -> Unit
) {
    val rememberedColors = remember {
        // Explicitly creating a new object here so we don't mutate the initial [colors]
        // provided, and overwrite the values set in it.
        colors.copy()
    }.apply { updateColorsFrom(colors) }
    val rippleIndication = rememberRipple()
    val selectionColors = rememberTextSelectionColors(rememberedColors)
    CompositionLocalProvider(
        LocalColors provides rememberedColors,
        LocalContentAlpha provides ContentAlpha.high,
        LocalIndication provides rippleIndication,
        LocalRippleTheme provides MaterialRippleTheme,
        LocalShapes provides shapes,
        LocalTextSelectionColors provides selectionColors,
        LocalTypography provides typography
    ) {
        ProvideTextStyle(value = typography.body1) {
            PlatformMaterialTheme(content)
        }
    }
}
```

이렇게 MaterialTheme 내부에서는 color, shape, typography이 compositionLocal로 정의되어 있다. 따라서 모든 하위 composable에서 local color, shape, typography를 `MaterialTheme.~`와 같이 간편하게 가져올 수 있는 것이다.

```kotlin
@Composable
fun MyApp() {
    // Provides a Theme whose values are propagated down its `content`
    MaterialTheme {
        // New values for colors, typography, and shapes are available
        // in MaterialTheme's content lambda.

        // ... content here ...
    }
}

// Some composable deep in the hierarchy of MaterialTheme
@Composable
fun SomeTextLabel(labelText: String) {
    Text(
        text = labelText,
        // `primary` is obtained from MaterialTheme's
        // LocalColors CompositionLocal
        color = MaterialTheme.colors.primary
    )
}
```

## 🔗 특정 범위에 다른 compositionLocal값 적용

> `CompositionLocal.current`는 <span style = "background-color:#fff5b1">composition 범위가 지정된 부분의 상위 요소가 제공하는 가장 가까운 값을 가져오게 된다.</span> 

따라서 원하는 범위에 compositionLocal을 새로 지정하게 되면 그 하위 요소들에게 가장 가까운 값은 새로 지정한 compositionLocal 값이 되기 때문에 지정한 값을 사용할 수 있게 되는 것이다.

## 🔗 compositionLocal 생성!

1. compositionLocal 생성하기

compositionLocal을 만드는 데는 두가지 API가 존재한다.

- <span style = "background-color:#fff5b1">compositionLocalOf</span>
  - recomposition 중 값이 바뀐다면 `current`를 읽는 콘텐츠만 재구성된다.
  - 변경이 잦은 상태를 다룰 때 이용하는 것이 좋다!

- <span style = "background-color:#fff5b1">staticCompositionLocalOf</span>
  - compositionLocalOf와 달리 compose에 의해 추적되지 않는다.
  - recomposition 중 값이 바뀌면 `current`를 읽는 위치만 아니라 <span style = "background-color:#fff5b1">CompositionLocal이 제공된 content 람다 전체가 재구성된다.</span>

만약 <span style = "background-color:#fff5b1">compositionLocal에 제공된 값이 변경될 가능성이 거의 없거나 변경되지 않는다면</span> `staticCompositionLocalOf`를 사용해 성능을 향상시키자!

두 생성 방식의 작동 방식이 조금 헷갈린다면 [해당 게시글](https://www.charlezz.com/?p=46403)의 영상과 코드를 참고하자!

```kotlin
// LocalElevations.kt file

data class Elevations(val card: Dp = 0.dp, val default: Dp = 0.dp)

// Define a CompositionLocal global object with a default
// This instance can be accessed by all composables in the app
val LocalElevations = compositionLocalOf { Elevations() }
```

`CompositionLocalProvider`를 사용해 compositionLocal 인스턴스에 값을 바인딩한다. `provides`를 통해 key와 value를 연결해주며, key에는 제공하는 compositionLocal을 value에는 key에 접근했을 때 실제 가져와지는 compositionLocal 값을 넣어준다.

```kotlin
@Composable
fun CompositionLocalExample() {
    MaterialTheme { // MaterialTheme sets ContentAlpha.high as default
        Column {
            Text("Uses MaterialTheme's provided alpha")
            CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
                Text("Medium value provided for LocalContentAlpha")
                Text("This Text also uses the medium value")
                CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.disabled) {
                    DescendantExample()
                }
            }
        }
    }
}

@Composable
fun DescendantExample() {
    // CompositionLocalProviders also work across composable functions
    Text("This Text uses the disabled alpha now")
}
```

여기서 `LocalContentAlpha provides ContentAlpha.medium`를 본다면 LocalContentAlpha값이 ContentAlpha.medium으로 제공된다는 뜻이다.

만약 compositionLocal의 현재 값에 접근하고 싶다면 위에서 한 번 언급했던 것처럼 `current`를 이용하면 된다.

```kotlin
@Composable
fun FruitText(fruitSize: Int) {
    // Get `resources` from the current value of LocalContext
    val resources = LocalContext.current.resources
    val fruitText = remember(resources, fruitSize) {
        resources.getQuantityString(R.plurals.fruit_title, fruitSize)
    }
    Text(text = fruitText)
}
```

## 🔗 compositionLocal의 과도한 사용 ⛔️

compositionLocal을 과도하게 사용하는 것은 문제가 될 수 있다. 

1. `CompositionLocal`은 <span style = "background-color:#fff5b1">컴포저블 동작을 추론하기 어렵게 한다.</span>

- compositionLocal은 <span style = "background-color:#fff5b1">암시적으로 컴포지션을 통해 데이터를 전달하는 도구</span>이기 때문에 어떤 값이 적용되고 있는지를 한눈에 파악하기 어렵다. 따라서 예상대로 컴포저블이 동작하지 않을 수 있다.

2. 컴포지션의 모든 부분에서 변경될 수 있으므로 종속 항목에서의 디버깅이 어렵다.

- 컴포지션에 적용된 값을 current를 통해 확인해야 하므로 디버깅하는 과정이 번거로울 수 있다.

## 🔗 compositionLocal 사용 여부 결정

1. `CompositionLocal`에는 <span style = "background-color:#fff5b1">적절한 기본값이 있어야 한다.</span>

- 기본값이 없다면 테스트나 해당 compositionLocal을 사용하는 composable을 미리 볼 때 문제가 발생할 수 있다.

2. <span style = "background-color:#fff5b1">모든 하위 요소에 사용할 때 적합하다.</span>

- 예를 들어 viewModel을 compositionLocal로 사용해 모든 컴포저블이 viewModel을 참조할 수 있도록 한다면?
  - UI 트리 아래의 모든 컴포저블이 viewModel에 관해 알 필요는 없으므로 좋지 않은 방식이다.