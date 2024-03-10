---
title: "[Compose] custom theme 구성하기(2)"
excerpt: "compositionLocal을 활용한 custom theme 구성"
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
지난 포스팅 [[custom theme 구성하기(1): compositionLocal에 대해서 먼저 알자!]](https://915dbfl.github.io/compose/custom_theme/)에서 custom theme을 구성할 때 사용되는 `CompositionLocal`에 대해 깊이 알아보았다. 그렇다면 이제 이를 활용해 custom theme을 직접 구성해보자!

우선 필자는 custom theme을 구성하기 이전 기존 `color.xml`, `font_style.xml` 등 xml에 정의된 resource에 직접 접근하는 방식으로 코드를 구성했다. 

```kotlin
        modifier = modifier
            .border(
                width = 1.dp,
                color = colorResource(id = R.color.gray_2), // 이렇게 직접 resource에 접근
                shape = RoundedCornerShape(10.dp)
            )
```

하지만 매번 <span style = "background-color:#fff5b1">resoure의 접근하는 것은 비용이 많이 들고, 현재 진행하는 프로젝트 속 사용하는 color와 typography가 xml에 정의되어 있기 때문에</span> custom theme을 정의해 적용하기로 했다.

만약, darkMode / lightMode를 지원하는 경우라면 더욱 custom theme을 활용하는 것이 효과적이다. 큰 수고를 들이지 않고 darkMode와 lightMode에 해당하는 스타일을 지정할 수 있기 때문이다!!

그렇다면 본격적으로 custom theme을 만들어보자! 그 과정은 [Custom design systems in Compose(안드로이드 공식 문서)](https://developer.android.com/jetpack/compose/designsystems/custom)에 정말 잘 정리되어 있으며, 필자도 해당 글을 통해 custom theme을 정의해 활용했다!

<br>

# 🔗 Material Theming extend하기 1: 단순 extend

우선 완전히 custom theme을 구성하기 전에 공식 문서를 보면 Material Theme을 extend하는 방식이 나온다. 

해당 방식은 정말 간단하다. Material Theme에서 제공하는 `Color`, `Shape`, `Typography`를 활용해 extend를 적용하면 된다.

```kotlin
val Typography.heading2B: TextStyle
    @Composable get() = TextStyle(
        fontWeight = FontWeight.Bold,
        fontSize = 28.sp,
        lineHeight = 39.2.sp
    )

val Typography.heading2M: TextStyle
    @Composable get() = TextStyle(
        fontWeight = FontWeight.Normal,
        fontSize = 28.sp,
        lineHeight = 39.2.sp
    )

```

해당 방식은 단순히 몇 개의 테마 값만을 추가할 때 정말 간편하다. 하지만 공식 문서에서는 해당 방식에 대해 다음과 같이 설명을 덧붙이고 있다.

> If you have multiple themes, it’s better to define a class with new properties instead.

만약 여러 theme(ex: darkTheme, lightTheme)을 제공한다면 직접 theme을 정의하는 것이 좋다.

<br>

# 🔗 CompositionLocal 활용해 custom theme 활용

compositionLocal을 만들고 적용하는 것까지는 custom theme을 구성하는 데 공통적으로 들어간다. 이렇게 구성한 custom theme을 material theme과 함께 사용할지, 말지에서 차이가 나는 것이다!

그렇다면 우선 custom theme을 구성하자!

## 🔗 필요한 data class 정의

우선 추가할 값들을 가지는 data class를 정의한다. 여기서는 `Color`, `Typography`, `Elevation`, `Shape` 등 원하는 항목의 값들을 자유롭게 정의하면 된다.

```kotlin
@Immutable
data class LGTMColor(
    val yellow: Color,
    val blue: Color,
    val red: Color,
    val green: Color,
    ...
)

```

여기서 적용하는 <span style = "background-color:#fff5b1">@Immutable</span> 어노테이션은 컴포즈에게 해당 object가 `immutable`하므로 <span style = "background-color:#fff5b1">단순 사용만 하며 recomposition이 불필요하다</span>는 것을 전달해준다.

## 🔗 staticCompositionLocalOf로 compositionLocal 생성

```kotlin
val LocalColorPalette = staticCompositionLocalOf {
    LGTMColor(
        yellow = Color.Unspecified,
        blue = Color.Unspecified,
        red = Color.Unspecified,
        ...
    )
}
```

compositionLocal을 다루는 게시글에서 알아봤듯이 <span style = "background-color:#fff5b1">theme의 값은 변경될 가능성이 낮거나 거의 변경되지 않기 때문에</span> `staticCompositionLocal`을 활용해 생성함으로써 <span style = "background-color:#fff5b1">성능을 향상시키자.</span>

생성할 때는 default값을 넣어주고 실제로 활용될 값은 적용할 때 넣어준다.

## 🔗 생성한 compositionLocal 적용

생성한 compositionLocal을 적용할 때는 <span style = "background-color:#fff5b1">CompositionLocalProvider</span>을 활용한다. <span style = "background-color:#fff5b1">provides</span>를 통해 실제 적용할 값을 바인딩시키자!

```kotlin
@Composable
fun LGTMTheme(
    content: @Composable () -> Unit
) {
    val lgtmColorPalette = LGTMColor(
        yellow = Color(0xFFfac53d),
        blue = Color(0xFF1d74f5),
        red = Color(0xFFfe504f),
        ...
    )

    val lgtmTypography = LGTMTypography(
        heading1B = TextStyle(
            fontWeight = FontWeight.Bold,
            fontSize = 32.sp,
            lineHeight = 44.8.sp
        ),
        ...
    )

    // 해당 부분의 코드가 다양하게 변경된다.
    CompositionLocalProvider(
        LocalColorPalette provides lgtmColorPalette,
        LocalTypography provides lgtmTypography,
        content = content
    )
}
```

## 🔗 생성한 custom theme 접근을 위한 object 생성

이제 이렇게 생성한 compositionLocal theme 값들을 코드에서 쉽게 활용하기 위해 다음과 같은 obejct 클래스를 만든다! theme은 프로젝트 전체에서 하나의 인스턴스만 존재하면 되기 때문에 `object`를 활용해 `singleton`으로 구성한다.

```kotlin
object LGTMTheme {
    
    val colors: LGTMColor
        @Composable
        @ReadOnlyComposable
        get() = LocalColorPalette.current

    val typography: LGTMTypography
        @Composable
        @ReadOnlyComposable
        get() = LocalTypography.current

}
```

여기서 우리가 생성한 <span style = "background-color:#fff5b1">CompositionLocal에 .current를 통해 현재 적용된 compositionLocal 값을 가져와</span> 변수에 넣어준다. 이렇게 되면 우리가 지정한 값들을 활용할 수 있게 되는 것이다.

위에서 정의한 `colors`, `typography`는 <span style = "background-color:#fff5b1">object 내에서 읽기만을 수행하며, 해당 값을 불러 사용하는 composable에서도 해당 값에 대해 읽기만 수행하므로</span> `@ReadOnlyComposable` 어노테이션을 붙여 코드의 효율을 높인다.

## 🔗 object를 통해 theme 활용하기

위와 같이 object까지 구성했다면 이제 마음껏 활용할 차례다! 활용도 간단하다!

```kotlin
 ModalBottomSheetLayout(
        sheetState = bottomSheetState,
        sheetShape = RoundedCornerShape(
            topStart = 20.dp,
            topEnd = 20.dp
        ),
        scrimColor = LGTMTheme.colors.transparent_black, // Theme.colors와 같이 접근해 활용하면 된다.
        ....
 )
```

그리고 우리가 처음 Screen Composable을 호출할 때 겉에 테마를 감싼 것처럼 우리가 커스텀한 Theme을 활용하면 된다.

```kotlin
        LGTMTheme {
            SuggestionDetailScreen()
        }
```

<br>

# 🔗 Material Theming extend하기 2: Material Theme을 감싸는 새로운 테마 정의

그렇다면 Material Theme을 extend하는 두 번째 방식을 살펴보자! 위에서 생성한 custom theme을 <span style = "background-color:#fff5b1">Material Theme을 감싸는 형태로 정의</span>해 material theme에서 제공하는 속성도 사용하고 새로운 속성도 정의하는 방식이다.

```kotlin
    CompositionLocalProvider(LocalExtendedColors provides extendedColors) {
        MaterialTheme(
            /* colors = ..., typography = ..., shapes = ... */
            content = content
        )
    }
```

간단하다! `CompositionLocalProvider`이 MaterialTheme을 감싸는 형태로 구성하면 된다. 그렇다면 우리는 Material Theme에서 제공하는 값과 우리가 새롭게 정의한 값을 함께 사용할 수 있게 된다.

즉, `LGTMTheme.colors`를 통해서 새로 정의한 값을 가져올 수 있고, `MaterialTheme.colors`를 통해 material theme에서 제공하는 값을 가져와 활용할 수도 있다.

<br>

# 🔗 Material Theme 대체하기

<span style = "background-color:#fff5b1">기존 Material Theme에 정의된 값을 재정의 할 수도 있다.</span> 그럴 경우, custom theme을 만드는 방식과 적용하는 것도 위와 동일하다.

```kotlin
    CompositionLocalProvider(LocalExtendedColors provides extendedColors) {
        MaterialTheme(
            /* colors = ..., typography = ..., shapes = ... */
            content = content
        )
    }
```

단 사용할 때 주의해야 한다. <span style = "background-color:#fff5b1">기존 값이 아닌 직접 정의한 값을 사용하고자 한다면 ProvideTextStyle을 활용해 값을 확실히 지정해주는 것이 좋다.</span>

```kotlin
@Composable
fun ReplacementButton(
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    content: @Composable RowScope.() -> Unit
) {
    Button(
        shape = ReplacementTheme.shapes.component,
        onClick = onClick,
        modifier = modifier,
        content = {
            ProvideTextStyle( // 직접 ReplacementTheme 속에 있는 body typography를 지정해줌
                value = ReplacementTheme.typography.body
            ) {
                content()
            }
        }
    )
}
```

<br>

# 🔗 Custom Theme만을 활용하기

위에서 정의한 custom Theme을 다음과 같이 적용하면 된다.

```kotlin
    CompositionLocalProvider(
        LocalColorPalette provides lgtmColorPalette,
        LocalTypography provides lgtmTypography,
        content = content
    )
```

그렇다! `MaterialTheme`을 감싸는 코드를 제거해준다. 

여기서 잠깐! 우선 Material Theme이 제공하는 것들을 확인해보자!

1. `Colors`, `Typography`, `Shapes`
    - material theming systems
2. `ContentAlpha`
    - 텍스트 및 아이콘 강조를 위한 불투명 정도
3. `TextSelectionColors`
    - text / textField에 의해 사용되는 텍스트 선택에 대한 색상
4. `ripple` and `rippleTheme`

<span style = "background-color:#fff5b1">이러한 material theme이 우리가 활용하는 Material Component에 활용되고 있다.</span> 따라서 Material Component를 활용하고자 한다면 위의 요소들을 모두 대체해야 원하지 않는 결과값이 나오는 것을 예방할 수 있다.

하지만 몇 개의 system만을 대체해도 된다. 이 경우에는 component 안에서 원하지 않는 결과가 나오지 않도록 필요에 따라 값을 추가로 hardcoding 해야 할 수도 있다는 점에 유의하자!