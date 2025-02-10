---
title: "[Compose] ui test(2): synchronization 다루기 & ui test common pattern 살펴보기"
categories:
  - Compose
tag:
  - compose ui test
  - ui test
  - compose
  - synchronization
  - ui test common pattern

last_modifeid_at: 2025-02-10
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며

[이전 게시글: [Compose] ui test(1)](https://915dbfl.github.io/compose/compose-ui-test/)에서는 testing api와 compose ui test에서 중요한 `semantic`에 대해 다뤄 보았다. 이번 글에서 다룰 내용은 크게 다음 세 가지 이다.

1. `synchronization`
2. `compose ui test common pattern`
3. `debug test`


그렇다면 시작해보자!

<br>

# 🔗 synchronization

## 1. TestRule

우선 synchronization에 대해 다루기 전에 `TestRule`에 대해 짚고 넘어 가고자 한다.

> A [`TestRule`](https://junit.org/junit4/javadoc/4.12/org/junit/rules/TestRule.html) that allows you to test and control composables, either in isolation or in applications. Most of the functionality in this interface provides some form of test synchronization: the test will block until the app or composable is idle, to ensure the tests are deterministic.
> 
> 
> testRule은 composable을 테스트하고 제어할 수 있도록 한다. testRule 인터페이스에서 제공되는 대부분의 함수는 test synchronization을 제공한다.
>

test를 진행할 때 기본적으로 `testRule`을 사용한다. 그리고 이런 testRule 덕분에 test에서 synchronization을 활용할 수 있게 된다. 이러한 testRule 구현체는 여러 가지가 존재하기 때문에 상황에 맞는 구현체를 선택해 활용해야 한다.

1. `createComposeRule`
    - compose content에 대한 host를 자동으로 생성해준다. 따라서 <span style = "background-color:#fff5b1">activity 없이 ui component만 테스트할 때 유용하다.</span>
    - activity를 시작하지 않기 때문에 createAndroidComposeRule보다 테스트 실행 속도가 빠르다.
2. `crreateAndroidComposeRule`
    - <span style = "background-color:#fff5b1">특정 호스트(activity)를 지정할 때 사용한다.</span> Rule을 통해 지정한 Activity를 시작하고, test가 끝난 후에 해당 activity를 종료한다.
3. `createEmptyComposeRule`
    - test rule을 통해 어떤 Activity도 시작할 필요가 없을 때 사용한다. 따라서 기본적으로 setContent를 제공하지 않는다.
    - setContent를 활용하기 위해서는 별도의 뷰를 생성해 해당 뷰에서 제공되는 setContent를 활용해야 한다.

## 2. test synchronization?🤔

1. ui synchronization 활용하기 

    그렇다면 본격적으로 compose test의 동기화 매커니즘에 대해 다뤄보자! 우선 다음의 상황을 생각해보자!

    > ui에 필요한 데이터를 얻고 화면을 그리는데 대기시간이 10초 필요한 상황이다.

    이 경우, test를 할 때도 10초 이상이 걸릴까? 10초를 기다리고 싶지 않다면 어떻게 해야 할까?

    이러한 상황을 다루는 것이 바로 `test synchronization`이다! 이를 통해 테스트의 정확성과 속도를 어떻게 최적화하는지 살펴보자!

    ```kotlin
    @Test
    fun counterTest() {
        val myCounter = mutableStateOf(0) // State that can cause recompositions.
        var lastSeenValue = 0 // Used to track recompositions.
        composeTestRule.setContent {
            Text(myCounter.value.toString())
            lastSeenValue = myCounter.value
        }
        myCounter.value = 1 // The state changes, but there is no recomposition.

        // Fails because nothing triggered a recomposition.
        assertTrue(lastSeenValue == 1)

        // Passes because the assertion triggers recomposition.
        composeTestRule.onNodeWithText("1").assertExists()
    }
    ```
    위에서 언급했듯이 <span style = "background-color:#fff5b1">testRule을 활용하면 compose test는 기본적으로 ui와 동기화 된다. assertion과 action이 호출된다면 test는 ui tree가 그려질 때까지 기다림으로써 동기화를 진행한다.</span>

    첫 번째 assertion의 경우, composeTestRule 밖에서 assertion이 진행된다. 이럴 경우, 변경된 값을 인식하고 이를 업데이트하기 위한 recomposition이 trigger되지 않는다. 따라서 lastSeenValue는 여전히 0을 나타내게 된다.

    <span style = "background-color:#fff5b1">하지만 두 번째 assertion의 경우, composeTestRule을 바탕으로 assertion이 진행된다. 따라서 기본적으로 ui synchronization이 진행되기 때문에 변경된 값이 인식되어 recomposition이 trigger된다.</span> 따라서 업데이트된 값인 1이 인식되게 되는 것이다.

2. test에서의 시간 관리

    compose ui test에서는 실제 시간 대신 <span style = "background-color:#fff5b1">가상 시계</span>를 사용하여 시간 관리를 진행한다. <span style = "background-color:#fff5b1">이를 통해 애니메이션이나 지연 시간이 존재할 경우, 가상 시간을 앞당김으로써 즉시 완료된 것처럼 처리할 수 있게 된다.</span>

    이를 통해 `test가 최적화` 된다. 1초 짜리 애니메이션이 있을 경우, 이를 기다리지 않고 시간을 앞당김으로써 테스트가 최대한 빠르게 실행될 수 있도록 하는 것이다. 

## 3. 자동 동기화 해제

`TestRule`을 활용할 때 자동으로 지원되는 동기화를 멈추거나 시간을 자체적으로 control할 경우도 분명 존재할 것이다.

이를 위해서는 다음의 코드를 활용할 수 있다.

```kotlin
composeTestRule.mainClock.autoAdvance = false
```

그리고 원하는 만큼 시간을 앞당길 수도 있다. 다음과 같이 시간 Frame을 앞당길 수도 있고, 특정 시간 이후로도 시간을 앞당길 수도 있다. 

```kotlin
composeTestRule.mainClock.advanceTimeByFrame()
composeTestRule.mainClock.advanceTimeBy(milliseconds)
```

## 4. Idle resources 등록 및 해제

다음의 상황을 생각해보자!

> 1. 네트워크 요청 결과를 바탕으로 ui를 그린다.
>
> 2. ui test를 진행한다.
>
> 3. 네트워크 요청이 완료되지 않아 테스트에 실패한다.

이처럼 <span style = "background-color:#fff5b1">비동기 작업이 ui에 영향을 미칠 경우</span>는 어떻게 테스트를 진행해야 할까?

Espresso에서 제공하는 Idling Resource와 비슷하게 compose ui test에서도 idle resource를 등록할 수 있는 기능을 제공한다. <span style = "background-color:#fff5b1">이를 통해 비동기 작업이 진행되는지 여부에 따라 상태가 결정되고 (busy or idle), test는 idle 상태가 될 때까지 기다린 후 진행될 수 있도록 한다.</span>

```kotlin
composeTestRule.registerIdlingResource(idlingResource)
composeTestRule.unregisterIdlingResource(idlingResource)
```

## 5. 수동으로 동기화 처리하기

기본적으로 동기화를 제공해주는 api를 활용한다면 동기화를 위한 별도의 작업이 필요하지 않을 수 있다. 하지만 만약 동기화 작업에 대한 커스텀이 필요하다면 `waitForIdle`, `advanceTimeUntil`을 활용할 수 있다.

우선 `waitForIdle`은 위에서 자동 동기화 해제와 관련해 언급했던 `autoAdvance`와 함께 사용된다. 

```kotlin
composeTestRule.mainClock.autoAdvance = true // Default
composeTestRule.waitForIdle() // Advances the clock until Compose is idle.

composeTestRule.mainClock.autoAdvance = false
composeTestRule.waitForIdle() // Only waits for idling resources to become idle.
```

다음과 같이 `advance = true`와 함께 사용한다면 idle 상태까지 시간을 앞당길 수 있게 된다. `advance = false`의 경우에는 시간을 앞당기지 못하기 때문에 `waitForIdle`을 사용할 경우, idle 상태까지 직접 시간 대기가 일어나게 된다.

또한 `advanceTimeUntil`을 활용한다면 특정 조건을 만족하는 시점까지 시간을 앞당길 수 있다.

```kotlin
composeTestRule.mainClock.advanceTimeUntil(timeoutMs) { condition }
```

여기서 말하는 특정 조건은 <span style = "background-color:#fff5b1">시간에 따라 영향을 받는 state와 관련된 조건이어야 한다.</span> 만약 시간과 관련된 조건이 아닌 단순 조건을 활용하고 싶다면 다음의 api들을 활용하자!

```kotlin
composeTestRule.waitUntil(timeoutMs) { condition }

// waitUntil helpers
composeTestRule.waitUntilAtLeastOneExists(matcher, timeoutMs)

composeTestRule.waitUntilDoesNotExist(matcher, timeoutMs)

composeTestRule.waitUntilExactlyOneExists(matcher, timeoutMs)

composeTestRule.waitUntilNodeCount(matcher, count, timeoutMs)
```

<br>

# 🔗 Common patterns

지금부터 공식 문서에 나와 있는 test common pattern들에 대해 정리해보고자 한다. 다음의 common pattern을 바탕으로 ui test를 어떻게 진행해야 하는지 감을 익혀보자!

## 1. Test in isolation

`ComposeTestRule`은 setContent를 통해 activity에 composable이 display될 수 있도록 한다. <span style = "background-color:#fff5b1">여기서 ui Test를 진행할 때는 각각의 테스트가 올바르게 캡슐화 되고, 독립적으로 작동하는지가 중요하며, 이는 ui 테스트를 더욱 쉽게 만든다.</span>

하지만 그렇다면 무조건 unit ui test만을 진행하라는 것은 아니다. 큰 범위의 ui test 역시 매우 중요하다.

## 2. Access the activity and resources after setting your own content

때때로 test를 진행하기 전에 `composeTestRule.setContent`를 통해 content를 셋팅해야 할 때가 있다. 

하지만 rule이 `createAndroidComposeRule`을 통해 생성이 되었다면, <span style = "background-color:#fff5b1">activity에서는 이미 onCreate를 진행할 때 setContent를 호출했기 때문에 추가로 setContent를 호출할 수 없게 된다.</span>

이를 위해서는 AndroidComposeTestRule을 `ComponentActivity`와 같이 <span style = "background-color:#fff5b1">빈 activity를 활용해 생성하면 된다.</span> ComponentActivity는 빈 액티비티이기 때문에 onCreate에서 setContent를 호출하지 않는다. 따라서 테스트 상황에서도 setContent를 호출하여 원하는 composable을 호출할 수 있게 된다.

```kotlin
class MyComposeTest {

    @get:Rule
    val composeTestRule = createAndroidComposeRule<ComponentActivity>()

    @Test
    fun myTest() {
        // Start the app
        composeTestRule.setContent {
            MyAppTheme {
                MainScreen(uiState = exampleUiState, /*...*/)
            }
        }
        val continueLabel = composeTestRule.activity.getString(R.string.next)
        composeTestRule.onNodeWithText(continueLabel).performClick()
    }
}
```
이를 활용하기 위해서는 다음의 두 가지 조건이 필요하다!

1. <span style =        "background-color:#fff5b1">ComponentActivity가 앱의 AndroidManifest.xml 파일에 추가되어 있어야 한다.</span>
2. 다음의 의존성 역시 추가해야 한다.
    ```kotlin
    debugImplementation("androidx.compose.ui:ui-test-manifest:$compose_version")
    ```

## 3. Custom semantics properties

custom semantic property를 정의하고, 이를 test에서 활용할 수 있다. 이를 위해서는
1. 필요한 `SemanticPropertyKey`를 새롭게 정의하고 
2. `SemanticPropertyReceiver`를 통해 새롭게 정의한 시맨틱을 이용 가능하도록 설정해야 한다.

```kotlin
// Creates a semantics property of type Long.
val PickedDateKey = SemanticsPropertyKey<Long>("PickedDate")
var SemanticsPropertyReceiver.pickedDate by PickedDateKey
```

이렇게 정의한 property는 semantics modifier에서 다음과 같이 활용이 가능하다.

```kotlin
val datePickerValue by remember { mutableStateOf(0L) }
MyCustomDatePicker(
    modifier = Modifier.semantics { pickedDate = datePickerValue }
)
```

그리고 test에서는 `SemanticsMatcher.expectValue`를 통해서 property 값에 대한 assertion을 진행할 수 있다.

```kotlin
composeTestRule
    .onNode(SemanticsMatcher.expectValue(PickedDateKey, 1445378400)) // 2015-10-21
    .assertExists()
```

> **여기서 주의할 점⭐️**
>
> <span style = "background-color:#fff5b1">custom semantics property는 주어진 finder나 matcher를 활용해 test하기 힘들 경우에만 사용해야 한다.</span>
>
> custom semantics를 통해 컬러나 폰트 사이즈와 같은 시각적인 property를 노출하는 방식은 권장되지 않는다. 이는 프로덕션 코드를 오염시킬 수 있으며, 잘못된 구현으로 인해 버그의 해결이 어려워 질 수 있기 때문이다. 
>
> 결론적으로 가능하다면 주어진 test api를 활용하자!

## 4. verify state restoration

activity나 프로세스가 재생성될 때 compose element의 state가 올바르게 저장되는지 확인하자. 이를 위해서는 `StateRestorationTester`를 활용해 손쉬베 진행할 수 있다.

<span style = "background-color:#fff5b1">해당 클래스는 composable이 재생성되는 상황을 시뮬레이션할 수 있도록 도와준다. 특히 "rememberSaveable"의 구현을 확인하는데 유용하다!</span>

```kotlin
class MyStateRestorationTests {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun onRecreation_stateIsRestored() {
        val restorationTester = StateRestorationTester(composeTestRule)

        restorationTester.setContent { MainScreen() }

        // TODO: Run actions that modify the state

        // Trigger a recreation
        restorationTester.emulateSavedInstanceStateRestore()

        // TODO: Verify that state has been correctly restored.
    }
}
```

## 5. Test different device configurations

안드로이드 앱은 다양한 조건에 맞춰 대응되어야 한다.(window sizes, locales, font sizes, dark / light theme 등) 이러한 조건들은 유저의 device level에 좌우되는데, 이 값을 `Configuration instance`를 통해 활용될 수 있다.

각 testing 환경에서는 특정 device level property가 구성되기 때문에 다양한 configurations 상황을 테스트하기가 어렵다.

이를 위해서는 `DeviceConfigurationOverride`를 활용할 수 있다. 이는 composable들에 대해 다양한 기기 구성을 적용해 시뮬레이션할 수 있도록 도와준다.

- `DeviceConfigurationOverride.DarkMode()`: 다크/라이트 테마 재정의
- `DeviceConfigurationOverride.FontScale()`: 시스템 글꼴 재정의
- `DeviceConfigurationOverride.FontWeightAdjustment()`: 글꼴 두께 재정의
- `DeviceConfigurationOverride.ForcedSize()`: 기기 크기와 상관없이 특정 사이즈 적용
- `DeviceConfigurationOverride.LayoutDirection()`: 레이아웃 방향(ltr, rtl) 재정의
- `DeviceConfigurationOverride.Locales()`: 로케일 재정의
- `DeviceConfigurationOverride.RoundScreen()`: 화면이 둥근지 재정의

```kotlin
composeTestRule.setContent {
    DeviceConfigurationOverride(
        DeviceConfigurationOverride.ForcedSize(DpSize(1280.dp, 800.dp))
    ) {
        MyScreen() // Will be rendered in the space for 1280dp by 800dp without clipping.
    }
}
```

다음과 같이 `DeviceConfigurationOverride` 함수를 상단에 두고, 설정하고자 하는 configuration에 대한 override를 파라미터로 전달한다. 만약 여러 개의 구성을 적용하고 싶다면 다음과 같이 [`DeviceConfigurationOverride.then()`](https://developer.android.com/reference/kotlin/androidx/compose/ui/test/DeviceConfigurationOverride#(androidx.compose.ui.test.DeviceConfigurationOverride).then(androidx.compose.ui.test.DeviceConfigurationOverride))을 활용하자.

```kotlin
composeTestRule.setContent {
    DeviceConfigurationOverride(
        DeviceConfigurationOverride.FontScale(1.5f) then
            DeviceConfigurationOverride.FontWeightAdjustment(200)
    ) {
        Text(text = "text with increased scale and weight")
    }
}
```

<br>

# 🔗 Debug tests

테스트를 디버그 하는 가장 기본적인 방법은 <span style = "background-color:#fff5b1">semantic tree를 확인하는 것</span>이다. semantic tree는 `composeTestRule.onRoot().printToLong()`를 통해서 출력할 수 있다.

```kotlin
Node #1 at (...)px
 |-Node #2 at (...)px
   OnClick = '...'
   MergeDescendants = 'true'
    |-Node #3 at (...)px
    | Text = 'Hi'
    |-Node #5 at (83.0, 86.0, 191.0, 135.0)px
      Text = 'There'
```

<br>

# 🔗 참고

- [https://developer.android.com/develop/ui/compose/testing/synchronization](https://developer.android.com/develop/ui/compose/testing/synchronization)
- [https://developer.android.com/develop/ui/compose/testing/common-patterns](https://developer.android.com/develop/ui/compose/testing/common-patterns)
- [https://developer.android.com/develop/ui/compose/testing/debug](https://developer.android.com/develop/ui/compose/testing/debug)