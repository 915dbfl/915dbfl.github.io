---
title: "[Compose] custom modifier"
excerpt: "원하는 기능이 modifier에 없다면??"
categories:
  - Compose
tag:
  - custom-modifier

last_modifeid_at: 2024-02-13
toc: true
toc_sticky: true
search: true
---

custom modifier를 만드는 데는 크게 두 가지 방법이 존재한다.

1. 기존 modifier를 가져와 `chaining`하는 방식
2. low level의 `Modifier.Node`를 활용해 조금 더 유연하게 custom하는 방식

이번 게시글에서는 1번 방식에 대해 다뤄보고자 한다!! chaining을 활용하는 데에도 크게 두 가지 방법이 존재한다.

1. 단순 chianing 방식
2. modifier factory 활용하는 방식

## 1. 단순한 chaining 방식
```
fun Modifier.clip(shape: Shape) = graphicsLayer(shape = shape, clip = true)
```
- 여기서 clip은 `graphicsLayer`를 사용해 구현되기 때문에 해당 블록으로 감싸 custom을 진행한다.

```kotlin
fun Modifier.myBackground(color: Color) = padding(16.dp)
    .clip(RoundedCornerShape(8.dp))
    .background(color)
```
- 자주 쓰이는 modifier function을 그룹으로 묶어 사용이 가능하다!

## 2. modifier factory 사용한 chaining 방식

<aside>
⛔️ 이전 compose 버전에서는 `composed {}`가 권장되었다. 하지만 lint rule이 제거 되면서 해당 방법은 권장되지 않음.

</aside>

```kotlin
@Composable
fun Modifier.fade(enable: Boolean): Modifier {
    val alpha by animateFloatAsState(if (enable) 0.5f else 1.0f)
    return this then Modifier.graphicsLayer { this.alpha = alpha }
}
```

<aside>
⛔️ modifier 체인을 끊지 않도록 주의해야 한다. this를 통해 기존 modifier를 가져오고, return을 통해 반환을 한다.

</aside>

## 2-1.  modifier factory를 통해 composition local 제공
- 만약 custom modifier를 통해 `CompositionLocal`의 defaut value를 제공하고자 한다면? 가능하다!!

```kotlin
@Composable
fun Modifier.fadedBackground(): Modifier {
    val color = LocalContentColor.current
    return this then Modifier.background(color.copy(alpha = 0.5f))
}
```

⛔️ 단, CompositionLocal 값은 call site에 결정된다. 즉, 사용될 때가 아니라 불려질 때 결정되므로 주의해야 한다!

```kotlin
@Composable
fun Modifier.myBackground(): Modifier {
    val color = LocalContentColor.current
    return this then Modifier.background(color.copy(alpha = 0.5f))
}

@Composable
fun MyScreen() {
    CompositionLocalProvider(LocalContentColor provides Color.Green) {
        // Background modifier created with green background
        val backgroundModifier = Modifier.myBackground()

        // LocalContentColor updated to red
        CompositionLocalProvider(LocalContentColor provides Color.Red) {

            // Box will have green background, not red as expected.
            Box(modifier = backgroundModifier)
        }
    }
}
```

## ⛔️ composable function modifiers는 절대 스킵되지 않는다?

composable function modifier는 <b>반환값이 있기 때문에</b> 절대 스킵되지 않는다!

이 말은 즉, recomposition이 일어날 때마다 call 되기 때문에 만약 recomposition이 빈번히 일어난다면 많은 비용이 들게 된다.

## ⛔️ composable function modifier 역시 composable 함수 내에서 호출되어야 한다?

```kotlin
val extractedModifier = Modifier.background(Color.Red) // Hoisted to save allocations

@Composable
fun Modifier.composableModifier(): Modifier {
    val color = LocalContentColor.current.copy(alpha = 0.5f)
    return this then Modifier.background(color)
}

@Composable
fun MyComposable() {
    val composedModifier = Modifier.composableModifier() // Cannot be extracted any higher
}
```

즉, composable function modifier를 이용하게 된다면 hoisting이 제한된다. 하지만 non-composable modifier factory의 겨우 위 코드의 extractedModifier처럼 composable 밖까지 호이스팅이 가능해진다. 이 점을 참고하자!

<br>

## 📝 참고 자료
* <https://developer.android.com/jetpack/compose/custom-modifiers>