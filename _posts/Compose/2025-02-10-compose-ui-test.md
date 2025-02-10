---
title: "[Compose] ui test(1): compose ui test에서의 다양한 testing api와 semantic tree"
categories:
  - Compose
tag:
  - compose ui test
  - ui test
  - compose

last_modifeid_at: 2025-02-10
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며

weQuiz 서비스 리팩토링 기간을 가지며, ui test의 필요성을 느끼게 되었다. 이번 게시글에서는 compose에서 ui test를 진행하기 위해 필요한 사전 지식들 중 compose ui testing api와 semantic tree에 대해서 다뤄보고자 한다. 대부분의 내용들은 compose ui test 공식 문서를 참고하였다.

<br>

# 🔗 compose ui test에서의 key concepts

1. `semantics`
- <span style = "background-color:#fff5b1">ui / element에 의미를 부여한다.</span>
- semantics tree는 ui hierarchy와 함께 생성되게 된다.
- semantics를 활용해 compose test는 ui와 상호작용할 수 있게 된다.

2. `testing api`
- compose가 제공하는 다양한 test api들을 활용할 수 있다.
    - 이를 통해 element를 찾고, state와 properties에 대해 assertion을 만들고, user interactions을 시뮬레이션 하기 위해 수행하는 등의 api를 활용할 수 있게 된다.

3. `synchronization`
- 기본적으로 compose test는 ui와 동기화된다.
    - 즉, assertion이나 performing을 수행하기 전에 `ui가 안정된 상태가 될 때까지 기다린다.` 이를 통해 ui가 완전히 랜더링된 후 테스트가 진행되도록 보장한다.
- synchronization을 없애거나 약간의 커스텀을 진행할 수 있다.

4. interoperability
- 하이브리드 앱(view system + compose)에서는 test가 통합될 수 있다.
    - (compose 100%앱에서의 test를 진행하고 하기 때문에 현재 해당 부분에 대해서는 추가적인 내용을 다루고 있지 않다. [관련 공식 링크](https://developer.android.com/develop/ui/compose/testing/interoperability)를 참고하자!)

<br>

# 🔗 Testing cheat sheet

[Testing cheat sheet](https://developer.android.com/develop/ui/compose/testing/testing-cheatsheet) 공식 문서를 확인해보면 compose ui test 과정에서 많이 사용되는 api 목록을 확인할 수 있다. 참고하자!

<br>

# 🔗 Testing APIs

ui element와 상호 작용하는 방법은 크게 세 가지가 존재한다.

1. `Finders`
    시맨틱 트리에서 하나 이상의 노드를 선택할 수 있도록 한다. 이렇게 선택한 노드에 대해서는 assertion을 수행하거나, 특정 actions을 수행한다.
2. `Assertions`
    element가 존재하는지, 혹은 특정 속성을 가지고 있는지를 확인할 때 사용한다.
3. `Actions`
    클릭 이벤트와 같이 시뮬레이션을 위한 user event를 element에 주입한다.

> **[여기서 잠깐! `semanticsMatcher` 다루고 넘어가기!]**
>
> 여기서 몇 가지 API들은 시맨틱 트리에서 하나 이상의 노드들을 참조하기 위해 `semanticsMatcher`를 매개변수로 받는다. filtering과 유사하다 생각하자. 특정 조건에 만족하는 특정 노드들을 고르는 것이다.
> ```kotlin
fun hasParent(matcher: SemanticsMatcher): SemanticsMatcher
fun hasAnySibling(matcher: SemanticsMatcher): SemanticsMatcher
fun hasAnyAncestor(matcher: SemanticsMatcher): SemanticsMatcher
fun hasAnyDescendant(matcher: SemanticsMatcher):  SemanticsMatcher
```

그렇다면 이제 각각에 대해 조금 더 자세히 다뤄보도록 하자!

## 1. Finders

관려내서 어떤 api를 활용할 수 있는지 보기 위해서는 위에 걸어둔 cheat sheet 링크에 들어가보자. 여기서는 간단한 예제만을 다룬다.

1. 하나의 노드 선택하기

    ```kotlin
    composeTestRule.onNode(<<SemanticsMatcher>>, useUnmergedTree = false): SemanticsNodeInteraction

    // 실제 사용
    composeTestRule
        .onNode(hasText("Button")) // Equivalent to onNodeWithText("Button")
    ```

2. 여러 노드들 선택하기

    ```kotlin
    composeTestRule
        .onAllNodes(<<SemanticsMatcher>>): SemanticsNodeInteractionCollection

    // 실제 사용
    composeTestRule
        .onNode(hasText("Button")) // Equivalent to onNodeWithText("Button")
    ```

3. <span style = "background-color:#fff5b1">unmerged tree</span>

    몇몇 노드들은 semantics 정보들이 병합이 되는 경우가 있다. 가장 대표적인 예가 Button이다. 일반적으로 버튼 composable을 사용할 때 다음과 같이 내부적으로 Text composable을 함께 선언한다. 그리고 버튼에 대한 semantics 정보를 확인할 경우, Button에서 Text값을 가지고 있는 것을 확인할 수 있다. 이런 경우가 바로 하위 semantics 정보를 상위 composable에서 병합해 가지고 있는 경우이다.

    ```kotlin
    MyButton {
        Text("Hello")
        Text("World")
    }

    // printLog로 semantics tree 출력
    Node #1 at (...)px
    |-Node #2 at (...)px
    Role = 'Button'
    Text = '[Hello, World]'
    Actions = [OnClick, GetTextLayoutResult]
    MergeDescendants = 'true'
    ```

    이렇게 할 경우, Text `World` text를 가지는 노드를 선택했을 때 MyButton 노드가 선택되게 된다. 그렇다면 Text 노드 자체를 선택할 수는 없을까? 이럴 때 사용하는 것이 <span style = "background-color:#fff5b1">useUnmergedTree = true</span>이다.

    printLog를 통해 `unmergedTree`는 어떻게 생겼는지 확인해보자!

    ```kotlin
    composeTestRule.onRoot(useUnmergedTree = true).printToLog("TAG")

    // 결과
    Node #1 at (...)px
    |-Node #2 at (...)px
    OnClick = '...'
    MergeDescendants = 'true'
        |-Node #3 at (...)px
        | Text = '[Hello]'
        |-Node #5 at (83.0, 86.0, 191.0, 135.0)px
        Text = '[World]'
    ```

    위 출력 결과와 달리 semantic 정보가 분리된 것을 확인할 수 있다. 따라서 다음과 같이 `useUnmergedTree = true`를 통해서 merge되징 않은 tree 상에서 원하는 노드를 선택할 수 있게 된다.

    ```kotlin
    composeTestRule
        .onNodeWithText("World", useUnmergedTree = true).assertIsDisplayed()
    ```

## 2. Assertions

하나의 노드 SemanticsNodeInteraction에 대해서 assertion를 호출할 수 있다. 

```kotlin
// Single matcher:
composeTestRule
    .onNode(matcher)
    .assert(hasText("Button")) // hasText is a SemanticsMatcher

// Multiple matchers can use and / or
composeTestRule
    .onNode(matcher).assert(hasText("Button") or hasText("Button2"))
```

그리고 node collection에 대해서 사용할 수 있는 assertion 연산 또한 존재한다.

```kotlin
// Check number of matched nodes
composeTestRule
    .onAllNodesWithContentDescription("Beatle").assertCountEquals(4)
// At least one matches
composeTestRule
    .onAllNodesWithContentDescription("Beatle").assertAny(hasTestTag("Drummer"))
// All of them match
composeTestRule
    .onAllNodesWithContentDescription("Beatle").assertAll(hasClickAction())
```

## 3. Actions

노드에 특정 `액션을 주입` 하기 위해서 <span style = "background-color:#fff5b1">perform...()</span> 함수를 사용할 수 있다.

```kotlin
composeTestRule.onNode(...).performClick()

// 사용할 수 있는 다양한 perform 함수
performClick(),
performSemanticsAction(key),
performKeyPress(keyEvent),
performGesture { swipeLeft() }
```

<br>

# 🔗 semantics in compose

우선 `semantic`가 무엇일까부터 짚고 넘어가자!

## 1. semantic?🤔

semantic의 뜻은 `의미론적`이다. 뜻을 통해서도 알 수 있듯이 <span style = "background-color:#fff5b1">시맨틱은 ui 요소에 의미를 부여하는 역할을 한다.</span> 그리고 compose에서는 이러한 시멘틱의 계층 구조를 나타내는 semantics tree가 존재한다.

composition은 하나의 tree이며 composable들을 실행함으로써 생성된다. 이때 composition 바로 옆에 병렬적으로 tree가 하나 더 존재하게 되는데 그것이 바로 semantics tree가 된다. 

기본적으로 compose foundation과 material library를 활용해 composable과 modifier를 구성하면 자동적으로 semantics tree가 채워지게 된다. 즉, compose에서 제공하는 composable을 사용하면 기본적으로 semantic 정보들이 채워지게 되는 것이다. 하지만 만약 low level의 composable을 새롭게 구성하게 된다면 그에 대한 semantics를 직접 제공해주어야 한다.

## 2. compose에서는 semantic이 왜 중요할까?🤔

<span style = "background-color:#fff5b1">compose에서 semantic이 중요한 이유</span>를 알기 위해서는 viewSystem과 compose에서의 ui 차이점을 살펴봐야 한다.

1. view system ui
- <span style = "background-color:#fff5b1">모든 ui 요소는 view의 인스턴스</span>이며, ui hierarchy를 통해 관리가 된다.
- 하나의 뷰는 직사각형의 공간을 차지하고 identifier / poisition / margin / padding과 같은 속성을 가지게 된다.

    > <span style = "background-color:#fff5b1">모든 ui가 view에 속하기 때문에 테스트시 각 view의 id와 속성을 적극적으로 활용할 수 있다.</span>

2. compose ui
- <span style = "background-color:#fff5b1">ui hierarchy에서 "일부 composable만" ui를 방출한다.</span>
- 어떤 composable은 단순히 다른 composable을 조합하거나 레이아웃을 구성하는 역할을 한다.
    - 즉, view system처럼 모든 것이 view인 것이 아니라, ui를 그리는 composable과 아닌 coposable이 혼재되어 있다. 
- 또한 composable은 view와 같이 명확한 id나 속성을 가지고 있지 않는다.

    > compose에서는 view의 id와 같은 직접적인 속성을 사용하는 것이 아니라 `Matcher`라는 개념을 사용해 ui 요소를 찾는다. 혹은 ui 요소의 의미적인 정보를 의미하는 `semantic`을 통해 접근성을 향상시키는 동시에 테스트에서도 유용하게 활용한다.

## 3. merged and unmerged semantics tree

testing api `Finder`에서 잠깐 다뤘듯이 semantic tree의 경우 merged / unmerged 두 가지가 존재한다. 그리고 semantic 정보의 병합은 다음 button과 같이 의미적으로 <span style = "background-color:#fff5b1">하나의 ui로 봐야 할때</span> 사용된다.

```kotlin
Button(onClick = { /*TODO*/ }) {
    Icon(
        imageVector = Icons.Filled.Favorite,
        contentDescription = null
    )
    Spacer(Modifier.size(ButtonDefaults.IconSpacing))
    Text("Like")
}
```
그렇다면 위의 요소에 대한 semantic 정보를 안드로이드 스튜디오 상에서 한번 확인해보자!

|화면|semantic 확인|
|:------:|:-------:|
|<img src = "/assets/images/button_screen.png">|<img src = "/assets/images/button_semantic_properties.png">|

semantic 정보는 android studio의 `layout inspector`를 통해서 확인할 수 있다. 위의 사진을 본다면 <span style = "background-color:#fff5b1">button의 merged semantics에 Text 값이 리스트 형태로 들어가 있는 것을 확인할 수 있다.</span>

이렇게 merged semantics를 제공하는 composable이나 modifier들을 본다면 다음의 코드를 가지고 있다.

```kotlin
modifier.semantics (mergeDescendants = true) {}
```

Button에서는 `clickable` 기능을 가지고 있는데 이 `clickable modifier` 내부적으로 `semantics modifier`를 포함하고 있다. 따라서 button의 하위 노드들의 semantic 정보가 merge되는 것이다.

> **[merged에 대해 짚고 넘어갈 부분!]**
>
> 여기서 말하는 병합이라는 것은 <span style = "background-color:#fff5b1">semantics tree내에서의 병합을 의미한다.</span> 즉, 해당 요소들이 완전히 합쳐져 이벤트까지 하나의 요소로써 받는 것이 아니라, 단순히 의미론적으로 병합을 나타낸다.
>
> 이를 통해서 test할 때는 각 요소들을 독립적으로 보지 않고 하나의 요소로 보며, 스크린리더 역시 병합된 요소 자체를 하나의 요소로 보게 된다.

그렇다면 버튼의 merged semantic property를 새롭게 추가해보고 실제로 merged semantic으로 들어가는지 확인해보자!

```kotlin
Box(
    modifier = Modifier.semantics(mergeDescendants = true) {
        contentDescription = "box with text"
    }
) {
    Text("Hello, World!")
}
```
다음과 같이 `contentDescription`을 merged semantic 정보로 새롭게 추가할 경우, 다음과 같은 결과를 확인할 수 있다.

|semantic 확인|
|:----:|
|<img src = "/assets/images/button_merged_semantic_addition.png">

## 4. 어떻게 merge가 되는 걸까?

composable이나 modifier가 semantics modifier를 통해 자식 노드들을 머지하겠다고 선언할 경우, 과연 이러한 merge 작업은 어떻게 일어나는 것일까?

각각의 semantics property는 `merging strategy`를 가지고 있다. 예를 들어 가장 대표적인 semantics property인 `ContentDescription`의 경우 자식들의 contentDescription value를 하나의 리스트에 더하는데, 이것이 바로 contentDescription에 적용된 merging strategy이다.

이러한 merging strategy에 대해서는 [SemanticsProperties.kt](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:compose/ui/ui/src/commonMain/kotlin/androidx/compose/ui/semantics/SemanticsProperties.kt;l=35?q=semanticsproperties&sq=) 속에 나와있는 mergePolicy를 통해 보다 구체적으로 확인할 수 있으니 참고하자!

## 5. merge 되지 않아야 하는 경우 다루기

<img src = "/assets/images/unmerged_example.png"/>

그렇다면 병합되지 않은 semantic으로 다루고 싶을 때는 어떻게 해야 할까? 다음의 ui에서는 하나의 item이 존재하고 해당 item을 클릭했을 때 item에 대한 상세 정보 페이지로 이동하게 된다. 그리고 오른쪽 북마크를 눌렀을 때는 별도의 북마크 이벤트를 처리해야 한다.

<span style = "background-color:#fff5b1">이때 스크린 리더의 경우, row가 클릭되는 것과 북마크가 클릭되는 것을 서로 다르게 인식해야 한다. 즉, semantics tree에서는 row와 북마크 아이콘이 독립적으로 존재해야 한다는 것이다.</span>

이렇게 만약 상위에서 하위의 semantic 병합을 제외하고 싶다면 하위 자체에서 `mergeDescendants = true` 값을 가지고 있으면 된다.

말로만 해서는 어렵다. 직접 확인해보자. 위에서 설명했듯이 button은 내부적으로 clickable을 가지고 있는데 이는 semantic modifier를 포함하고 있어 하위의 text 정보를 상위로 병합하게 한다. 그렇다면 여기에서 버튼으로부터 text의 semantic을 분리하기 위해서 text에 `mergeDescendants = true`를 설정해보자!!

```kotlin
// 기존
Button(
    onClick = onSubmitButtonClick,
    modifier = Modifier.fillMaxWidth(),
    enabled = isSignUpValid,
) {
    Text(
        text = when (isEditMode) {
            true -> stringResource(R.string.btn_sign_up_edit_user)
            false -> stringResource(R.string.btn_sign_up)
        },
    )
}

// text semantic 분리
Button(
    onClick = onSubmitButtonClick,
    modifier = Modifier.fillMaxWidth(),
    enabled = isSignUpValid,
) {
    Text(
        modifier = Modifier.semantics(mergeDescendants = true) {  }, // 자식이 자체적으로 해당 속성을 가지고 있도록 한다.
        text = when (isEditMode) {
            true -> stringResource(R.string.btn_sign_up_edit_user)
            false -> stringResource(R.string.btn_sign_up)
        },
    )
}
```

|분리 전|분리 후|
|:----:|:----:|
|<img src = "/assets/images/before_unmerging_semantic.png">|<img src = "/assets/images/after_unmerging_semantic.png">|

결과를 확인해본다면 button의 merged sematic을 확인해본다면 text에 대한 semantic이 모두 사라진 것을 확인할 수 있다!

<br>

# 🔗 참고

- [https://developer.android.com/develop/ui/compose/testing](https://developer.android.com/develop/ui/compose/testing)
- [https://developer.android.com/develop/ui/compose/testing/testing-cheatsheet](https://developer.android.com/develop/ui/compose/testing/testing-cheatsheet)
- [https://developer.android.com/develop/ui/compose/testing/semantics](https://developer.android.com/develop/ui/compose/testing/semantics)
- [https://developer.android.com/develop/ui/compose/testing/apis](https://developer.android.com/develop/ui/compose/testing/apis)