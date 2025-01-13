---
title: "[Compose] state를 어디로 hositing 해야 할까?"
excerpt: "state hositing을 적용할 때 고민해봐야 할 것들"
categories:
  - Compose
tag:
  - navigation compose
  - state hositing

last_modifeid_at: 2025-01-13
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며

`weQuiz` 서비스 리팩토링을 진행하면서 state hositing이 제대로 적용되어 있는지 확인하고, 개선하는 작업을 진행하게 되었다. 그 과정에서 state hositing을 어떠한 기준으로 적용해야 하는지 보다 명확하게 판단하기 위해 공식문서 [Where to hosit state](https://developer.android.com/develop/ui/compose/state-hoisting)를 학습하게 되었다.

그렇다면 공식문서에서는 state를 어디로 hositing 해야 한다고 말하고 있을까? 핵심 내용을 하나하나 살펴보자!

# 🔗 1. state hositing을 어디로 해야 할까?

우선 기본적인 <span style = "backgroudn-color:#fff5b1">state hoisting을 어디로 진행해야 할까?</span>를 살펴보자!

>  `lowest common ancestor`

즉, 해당 state를 필요로 하는 composable들의 가장 가까운 공통 조상으로 state를 hositing해야 한다. 이때 state는 최대한 state를 활용하는 위치에 가까이 두어야 한다는 점을 기억하자.

> [참고]
>
> 여기서 말하는 <span style = "background-color:#fff5b1">공통 조상이 꼭 composable이 되어야 하는 것은 아니다</span>. business logic과 state가 관련이 있다면 state를 composition 외부인 `viewModel`에 둘 수 있다.(이는 밑에서 자세히 다룬다.)

<br>

# 🔗 필요한 경우만 state hositing을 진행하자.

모든 경우에 state hoisting이 필요한 것은 아니다. 특정 state가 있을 때 해당 state를 다른 composable에서 필요로 하지 않는다면 state hoisting이 필요 없다. 

오히려 지나치게 state hoisting을 진행할 경우, <span style = "background-color:#fff5b1">불필요한 recomposition을 발생시킬 수도 있다.</span> 보통 state hositing을 진행하면 state를 상위에 배치하기 때문에 하위 composable 매개변수로 state를 전달하게 된다. 이때 state의 변경이 일어나게 되면 <span style = "background-color:#fff5b1">상위 composable 역시 state을 읽기 때문에 recomposition의 대상이 되게 된다.</span>

<br>

# 🔗 state 매개변수에 default값 지정하기

state hositing을 진행하게 되면 상위에서 state를 받아 활용하는 코드가 구성된다.

```kotlin
@Composable
private fun ConversationScreen(/*...*/) {
    val scope = rememberCoroutineScope()

    val lazyListState = rememberLazyListState() // State hoisted to the ConversationScreen

    MessagesList(messages, lazyListState) // Reuse same state in MessageList

    UserInput(
        onMessageSent = { // Apply UI logic to lazyListState
            scope.launch {
                lazyListState.scrollToItem(0)
            }
        },
    )
}
```

안드로이드 공식 문서의 코드를 한번 가져와봤다. 이처럼 ConversationScreen에 `lazyListState`를 구성해두고, 이를 하위 컴포저블인 MessageList와 UserInput에서 활용하고 있다. 

```kotlin
private fun MessagesList(
    messages: List<Message>,
    lazyListState: LazyListState = rememberLazyListState() // LazyListState has a default value
)
```

이때 MessageList의 생성자 부분을 살펴본다면 default값으로 `rememberLazyListState`를 지정해주고 있다. 이렇게 default 값을 지정해주는 것은 Compose에서 흔한 패턴이며, 이를 통해 <span style = "background-color:#fff5b1">composable의 재사용성을 높이고, 유연하게 만들 수 있게 된다. </span>

즉, 해당 composable을 다른 곳에서 사용할 때 state 관리를 신경쓰지 않고 편리하게 사용할 수 있게 된다. (이러한 패턴은 testing이나 previewing을 쉽게 하기 위해 많이 사용된다.)

```kotlin
@Composable
fun LazyColumn(
    modifier: Modifier = Modifier,
    state: LazyListState = rememberLazyListState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    verticalArrangement: Arrangement.Vertical =
        if (!reverseLayout) Arrangement.Top else Arrangement.Bottom,
    horizontalAlignment: Alignment.Horizontal = Alignment.Start,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    userScrollEnabled: Boolean = true,
    content: LazyListScope.() -> Unit
)
```
실제 `LazyColumn`의 내부 구현을 들어가본다면 `LazyListState`에 default값을 지정해둔 것을 확인할 수 있는 것처럼 현재 내부적으로 이러한 코드 패턴이 많이 활용되고 있다.

> [참고]
>
> 그렇다면 어디까지 `default값`을 지정하는 것이 좋을까? default 값을 지정하게 되면 test나 previewing할 때는 편리하지만, 너무 지나치게 활용할 경우,<span style = "bacgkround-color:#fff5b1">잘못된 값</span>이 내려왔을 때 판단을 하지 못할 수도 있다. 이를 잘 판단해서 활용하도록 하자!

<br>

# 🔗 4. state owner로 plain state holder를 구성하자⭐️ (추가 공부 필요)

composable에서 복잡한 ui logic을 가지고 있거나 여러 state를 관리해야 한다면 해당 역할을 별도의 `state holer`를 만들어 위임할 수 있다. <span style = "background-color:#fff5b1">즉, composable은 ui element을 emitting하는 역할에만 집중하고 state holder는 ui state의 관리와 이를 활용하는 ui logic에만 집중하며 관심사를 분리하는 것이다.</span>

실제로 위에서 다룬 `LazyListState`가 바로 plain state holder에 해당하는 예시이다. 그 안에 상세 구현을 한 번 살펴보자!

```kotlin
@OptIn(ExperimentalFoundationApi::class)
@Stable
class LazyListState @ExperimentalFoundationApi constructor(
    firstVisibleItemIndex: Int = 0,
    firstVisibleItemScrollOffset: Int = 0,
    private val prefetchStrategy: LazyListPrefetchStrategy = LazyListPrefetchStrategy(),
) : ScrollableState {

		// multi state 관리!
    internal var hasLookaheadPassOccurred: Boolean = false
        private set
    internal var postLookaheadLayoutInfo: LazyListMeasureResult? = null
        private set

    /**
     * The holder class for the current scroll position.
     */
    private val scrollPosition =
        LazyListScrollPosition(firstVisibleItemIndex, firstVisibleItemScrollOffset)

    private val animateScrollScope = LazyListAnimateScrollScope(this)
    
    // ui logic 관리!
    suspend fun animateScrollToItem(
        @AndroidXIntRange(from = 0)
        index: Int,
        scrollOffset: Int = 0
    ) {
        //...
    }

    /**
     *  Updates the state with the new calculated scroll position and consumed scroll.
     */
    internal fun applyMeasureResult(
        result: LazyListMeasureResult,
        isLookingAhead: Boolean,
        visibleItemsStayedTheSame: Boolean = false
    ) {
        //...
    }

}
```

다음과 같이 LazyListState state holder에서는 <span style = "background-color:#fff5b1">여러 state 및 ui logic을 캡슐화가 되어 있다.</span> 이 많은 state와 logic들이 LazyColumn 컴포저블 내부에 있다고 생각해보자. 생각만해도 복잡하지 않은가? 따라서 이렇게 state holder을 plain class로 분리함으로써 LazyColumn에서는 `LazyListState` 내부적으로 쓰이는 다른 state에 대한 관리에 신경을 쓰지 않아도 되는 것이다.

<br>

# 🔗 5. ViewModel로의 state hositing + Screen UI State

여기서 말하는 Screen UI State는 무엇일까? 위에서 state hositing에 다루면서 다음의 말을 잠깐 언급했다.

> business logic과 state가 관련이 있다면 state를 composition 외부인 viewModel에 둘 수 있다.

만약 `왜 state를 viewModel에 둘 수 있는거지?` 의문이 생긴다면 다음 viewModel의 정의 및 역할을 한번 짚고 넘어가자!

<details>

<summary>우리는 ACC viewModel을 왜 사용할까?</summary>

`AAC ViewModel`의 정의를 가져와봤다.

> The [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) class is a [business logic or screen level state holder](https://developer.android.com/topic/architecture/ui-layer/stateholders). It exposes state to the UI and encapsulates related business logic.
> 
> ViewModel은 비즈니스 로직, screen level의 state holde의 역할을 한다. state를 UI에 노출하며 관련된 비즈니스 로직을 캡슐화한다.
>

우리는 viewModel을 사용할 때 repository, usecase 등을 주입받아 사용한다. 이를 통해 데이터 / 도메인 레이어 등의 비즈니스 로직을 호출할 수 있게 된다.

그리고 viewModel에 uiDtata를 liveData / flow 등을 활용해 정의해둔다. 그 이유는 viewModel의 lifecycle이 activity보다 더 길어 configuration change와 같은 상황에서 uiData를 효과적으로 유지할 수 있기 때문이다.

</details>

<br>

즉, 위와 같이 business logic과 관련된 state는 viewModel까지 state hositing을 적용하게 된다. 이때 이렇게 viewModel에 정의되어 있는 state가 바로 여기서 말하는 `Screen Ui State`이다.

## 1. Screen Ui State가 필요한 이유 = viewModel로 state hositging이 필요한 이유

여기까지 왔다면 Screen Ui State가 필요한 이유에 대해 생각해볼 수 있다. 

> business logic과 state가 관련이 있다면 state를 viewModel에 둘 수 있다.

즉, 계속해서 말했듯이 viewModel에 존재하는 비즈니스 로직과 관련된 state라면 viewModel까지 state hoisting을 진행하여 Screen Ui State를 구성해야 한다.

> [참고]
>
> viewModel은 configuration change 상황에서도 uiData를 유지할 수 있도록 해준다. 만약 이러한 데이터가 `system-initiated process recreation` (ex: 메모리 부족으로 앱 강제 종료)와 같은 상황에서도 유지될 수 있도록 하고자 한다면 [SavedStateHandle](https://developer.android.com/reference/androidx/lifecycle/SavedStateHandle)을 활용하자!

<br>

## 2. parameter drilling

viewModel로 state를 hoisting한다면 이를 필요로 하는 composable에서 해당 state를 사용하기 위해서는 상위에서 하위로 계속해서 state를 전달해줘야 한다. (이렇게 data를 필요한 composable까지 게속해서 넘겨주는 것을 parameter drilling이라 한다.)

이 depth가 점점 깊어질수록 다들 '이게 괜찮은 방식인가?' 싶은 생각을 해봤을 것이다. 상위에서 data를 쭉 넘겨주다 보니 composable의 매개변수 개수도 점점 많아지고,,, 이게 과연 괜찮을까?

이에 대한 [공식 문서](https://developer.android.com/develop/ui/compose/state-hoisting#property_drilling)에 나온 내용을 가져와봤다.

> Even though exposing events as individual lambda parameters could overload the function signature, it maximizes the visibility of what the composable function responsibilities are. You can see what it does at a glance.
> 
> 
> 각 이벤트를 각각의 람다 파라미터로 만든다면 함수의 signature는 점점 커지고 복잡해질 수 있다. 하지만 이러한 방식은 composable이 어떠한 책임을 가지고 있는지 한눈에 파악하기 쉽게 한다.
> 

상위에서 하위로 파라미터를 넘겨주는 것 자체가 문제는 되지 않는다. 그 개수가 많아지더라도 <span style = "background-color:#fff5b1">꼭 필요한 매개변수</span>라면 함수 signature에 선언해주는 것이 오히려 해당 composable의 역할을 한눈에 파악할 수 있도록 해준다.

composable의 signature가 비대해진다면 우리가 가장 신경 써야 할 것은 <span style = "background-color:#fff5b1">composable 함수가 불필요한 state를 알고 있지 않을까?</span>이다.

> [참고]
>
> 하지만 만약 매개변수가 너무 많아지고, 성능문제가 발생한다면 공식 문서에서 설명하고 있는 것처럼 [defer reading of state](https://developer.android.com/develop/ui/compose/performance/bestpractices#defer-reads)를 적용할 수도 있다.

<br>

## 3. ui element state를 viewModel hoisting할 때 주의할 점: composition과 관련된 coroutineScope 사용하기

여기서 말하는 ui element state는 위에서 언급한 `LazyListState`와 같이 UI 요소 자체에서 필요로 하는 state를 의미한다. 이러한 state 역시 비즈니스 로직과 관련이 되게 된다면 viewModel로 hoisting을 진행할 수 있다.

viewModel로 해당 state를 hositing하게 된다면 viewModel 내부에서 해당 state가 제공하는 인터페이스들을 사용할 수 있게 된다. 잠깐 다시 한번 `LazyListState`의 내부 코드의 일부를 가져와 보자.

```kotlin
@OptIn(ExperimentalFoundationApi::class)
@Stable
class LazyListState @ExperimentalFoundationApi constructor(
    firstVisibleItemIndex: Int = 0,
    firstVisibleItemScrollOffset: Int = 0,
    private val prefetchStrategy: LazyListPrefetchStrategy = LazyListPrefetchStrategy(),
) : ScrollableState {
    suspend fun animateScrollToItem(
        @AndroidXIntRange(from = 0)
        index: Int,
        scrollOffset: Int = 0
    ) {
        animateScrollScope.animateScrollToItem(
            index,
            scrollOffset,
            NumberOfItemsToTeleport,
            density
        )
    }
}
```

위 코드를 본다면 `LazyListState`에서 제공하는 <span style = "background-color:#fff5b1">suspend interface</span>인 animateScrollToItem를 확인할 수 있다. 해당 함수는 `suspend`이기 때문에 별도의 코루틴 내부에서 호출해줘야 한다.

그렇다면 viewModel에 관련 state가 hositing 되어 있고, state에서 제공하는 suspend 함수를 `viewModelScope` 안에서 호출하게 된다면 어떻게 될까?

`IllegalStateException`이 발생한다. 그 이유는 <span style = "background-color:#fff5b1">viewModelScope는 Composition과 관련없는 scope이기 때문에 . 따라서 이를 해결하기 위해서 Composition에 scope된 CoroutineScope를 사용해야 한다. 아래 코드를 한번 살펴보자!</span>

(LazyListState가 아닌 DrawerState와 관련된 공식 문서 코드를 살펴보자!)
```kotlin
class ConversationViewModel(/*...*/) : ViewModel() {

    val drawerState = DrawerState(initialValue = DrawerValue.Closed)

    private val _drawerContent = MutableStateFlow(DrawerContent.Empty)
    val drawerContent: StateFlow<DrawerContent> = _drawerContent.asStateFlow()

		// composition과 관련된 coroutineScope 전달 받아 사용!
    fun closeDrawer(uiScope: CoroutineScope) {
        viewModelScope.launch {
            withContext(uiScope.coroutineContext) { // Use instead of the default context
                drawerState.close()
            }
            // Fetch drawer content and update state
            _drawerContent.update { content }
        }
    }
}

// in Compose
@Composable
private fun ConversationScreen(
    conversationViewModel: ConversationViewModel = viewModel()
) {
    val scope = rememberCoroutineScope()

    ConversationScreen(onCloseDrawer = { conversationViewModel.closeDrawer(uiScope = scope) })
}
```
위 코드와 같이 `Composition과 관련된 coroutineScop` 를 활용해서 viewModel에서 state의 suspend 함수를 호출해야 한다.

# 🔗 마무리하며

compose에서는 state에 따라 화면이 그려지고, state가 어떻게 관리되냐에 따라 성능도 크게 차이가 나기 때문에 state를 잘 관리하는 것이 매우 중요하다.

단순히 state hoisting을 적용하여 상위로 state를 끌어 올리는 것에서 나아가 복잡해지는 여러 state는 어떻게 관리해야 하는지, state를 전달할 때 성능을 조금 더 개선할 수 있는 방법은 또 무엇인지와 같이 다양한 관점에서 state를 효과적으로 관리할 수 있는 방법에 대해 학습해 볼 수 있었다.

추후 복잡해지는 여러 state 관리 및 [Follow best practices](https://developer.android.com/develop/ui/compose/performance/bestpractices#defer-reads)에서 나오는 state와 compose 성능 개선에 대해 조금 더 학습을 진행해봐야겠다!