---
title: "[Flow / Channel] Flow, Channel 쓰임새를 알고 잘 활용하기"
excerpt: "StateFlow vs SharedFlow vs Channel"
categories:
  - Android
tag:
  - Android
  - Flow
  - Channel
  - StateFlow
  - SharedFlow
  - Channel

last_modified_at: 2024-03-10
toc: true
toc_sticky: true
search: true
---

## 🔗 들어가기 전

compose로 작업을 진행하던 중, state를 효과적으로 관리하고자 MVI 패턴에 대해 학습하였다. orbit과 같은 외부 라이브러리를 사용하지 않고 직접 MVI 패턴을 적용할 때 <span style = "background-color:#fff5b1">event, state, sideEffect를 각각 SharedFlow, StateFlow, Channel</span>로 구현하는 코드를 보게 되었다.

이번 게시글을 통해서는 flow와 channel에 대해 다루고 어떠한 특징으로 인해 event, state, sideEffect가 각각의 구현 방식이 맞는지를 파악해보고자 한다. 해당 내용에 대해 정말 잘 정리된 게시글을 발견해 이를 토대로 글을 정리해볼 것이다.

- [Shared flows, broadcast channels](https://elizarov.medium.com/shared-flows-broadcast-channels-899b675e805c)

## 🔗 Channel의 탄생

coroutine이 등장하면서 공유 state에 대한 관리가 필요해졌다. 이를 위해 `channel`이라는 개념이 등장해 코투린 간 통신 기본 요소로 추가되었다.

channel은 일대일, 일대다, 다대일, 다대다 통신을 지원하며, 채널로 전송된 모든 값은 한 번만 수신된다.

![channel](/assets/images/channel.jpeg)

chaneel이 등장한 후, 사람들은 channel과 coroutine을 활용해 다양하게 데이터를 변환해서 사용했다. 다음의 과정을 살펴보자!

![channel filtering](/assets/images/channel_filtering.jpeg)

그렇다면 비동기 과정에서의 다양한 데이터 변환을 channel을 활용해 어떻게 구현해야 했을까? 만약 data에 특정 작업을 진행한 결과를 얻고 싶을 때는 두 개의 channel을 활용해 <span style = "background-color:#fff5b1"> 하나는 send, 하나는 receive로 활용해 중간에서 data 처리 작업을 진행했다.</span> 따라서 <span style = "background-color:#fff5b1">중간 처리를 위해서도 별도의 coroutine이 필요했다.</span>

코드로 어떻게 사용할 수 있는지 확인해보자!

```kotlin
class SingleShotEventBus {
    private val _events = Channel<Event>()
    val events = _events.receiveAsFlow() // expose as flow

    suspend fun postEvent(event: Event) {
        _events.send(event) // suspends on buffer overflow
    }
}
```

## 🔗 Flow의 등장

flow의 등장으로 <span style = "background-color:#fff5b1">data의 emit, transform, collect이 하나의 coroutine에서 가능</span>해지면서 위 과정이 정말 간편해졌다!

![flow](/assets/images/flow.jpeg)

flow에서는 emission과 collection이 서로 다른 coroutine에서 진행될 때만 동기화 처리를 진행하면 된다.

## 🔗 Flow = cold

`flow {}` 즉, flow 빌더를 통해 생성한 flow는 <span style = "background-color:#fff5b1">cold flow</span>이다.

여기서 말하는 `cold`는 어떤 것을 의미하는 걸까? 모바일 캠프에서 멘토님께서 설명하신 방식이 나는 cold라는 개념을 가장 명확히 설명할 수 있는 말 인 것 같다. cold란 `파이프 속 얼음`을 생각하면 된다. 파이프에 얼음이 막혀 있을 때 얼음을 깨면 이전에 있던 얼음이 쭉쭉 나오는 것처럼 flow에서 cold도 마찬가지다.

<span style = "background-color:#fff5b1">flow는 collect를 한 순간 이전의 값들이 우수수 들어오게 된다.</span> 따라서 collect 이전에는 아무런 상태를 가지지 않는다.

> 그렇다면 잠깐! channel은 cold일까? hot일까? channel은 hot이다. hot은 구독한 순간, 그 이후의 값들만 들어오는 것을 말한다.

그럼 <span style = "background-color:#fff5b1">user action이나 event, state update</span>와 같은 것을 단순 flow 다뤄도 될까? 우선 해당 값들이 가져야 하는 특징이 있다.

1. <span style = "background-color:#fff5b1"> 구독자가 있든 없든 계속해서 operate되어야 한다.</span>
2. <span style = "background-color:#fff5b1">여러 개의 observer를 지원해야 한다.</span>

따라서 `user-action`, `event`, `state update`와 같은 것들은 <span style = "background-color:#fff5b1">hot</span>으로 다뤄야 한다.

## 🔗 hot flow인 Shared flow의 등장

sharedFlow는 <span style = "background-color:#fff5b1">hot</span> flow이다. 따라서 구독자가 있든 없든 값을 emit한다.

![sharedflow](/assets/images/sharedflow.jpeg)

sharedFlow의 특징을 정리해보자!
- collect하고 구독자들은 모두 <span style = "background-color:#fff5b1">같은 순서의 값들을 받는다.</span>
- <span style = "background-color:#fff5b1">구독 이후</span>의 값들을 차례대로 받는다. (like `boardcast channel`)
- buffer가 존재하며, buffer가 꽉 찼을 때 emitter가 buffer에 자리가 날 때까지 잠시 멈춘다.
- 구독하고 이전 값 1회 반복(repeat), buffer가 꽉 찼을 때 처리 방식 변경 (BufferOverflow), extraBufferCapacity 등과 같이 다양한 파라미터를 이용해 설정을 변경할 수 있다.

```kotlin
class EventBus {
    private val _events = MutableSharedFlow<Event>() // private mutable shared flow
    val events = _events.asSharedFlow() // publicly exposed as read-only shared flow

    suspend fun produceEvent(event: Event) {
        _events.emit(event) // suspends until all subscribers receive it
    }
}
```
위 코드를 보면 sharedFlow를 emit할 때 `suspend`를 활용했다. 왜일까?

모든 구독자가 sharedFlow에 대해 동일한 순서의 값을 받는다. 따라서 sharedFlow를 emit할 때는 <span style = "background-color:#fff5b1">해당 구독자들이 그 값을 받을 때까지 suspend가 된다.</span>

## 🔗 최신 데이터만 가지고 있는 stateFlow의 등장

buffer overflow를 처리하는 가장 인기있는 방식은 오래된 데이터를 버리고, <span style = "background-color:#fff5b1">최신의 데이터만 유지하는 것이다.</span> 그 역할을 하는 것이 `stateFlow`이다.

```kotlin
class CounterModel {
    private val _counter = MutableStateFlow(0) // private mutable state flow
    val counter = _counter.asStateFlow() // publicly exposed as read-only state flow

    fun inc() {
        _counter.update { count -> count + 1 } // atomic, safe for concurrent use
    }
}
```
stateFlow는 최신의 data 하나만을 가지고 있는다. 따라서 sharedFlow와 다르게 값을 업데이트할 때도 굳이 suspend를 할 필요가 없다. 어차피 하나의 값만을 들고 있기 때문이다.

## 🔗 그렇다면 기존의 channel은 어디에서 활용할 수 있을까?

channel은 <span style = "background-color:#fff5b1">오직 한번만 실행되어야 하는 event를 처리할 때</span> 활용될 수 있다. 이러한 이벤트들은 구독자가 있을 수도 있고, 없을 수도 있다. 또한, 이전 event들을 구독자가 생길 동안 가지고 있어야 할 수도 있다. 

sharedFlow와 Channel의 차이를 다시 한 번 짚어보자!
1. `SharedFlow`
- zero or more 구독자 존재
- 구독자가 없다면 event는 drop된다.
  - 구독자가 있으면 무조건 처리가 되며, 구독자가 없으면 아무 것도 처리가 되지 않게 된다.

2. `Channel`
- 하나의 구독자가 존재한다.
- 구독자가 없을 경우, buffer가 가득 차면 event post를 멈추고 구독자를 기다린다.
  - post된 이벤트는 drop되지 않는다.

## 🔗 MVI에서 SharedFlow, StateFlow, Channel의 활용

우선 MVI에서 우리가 다뤄야 하는 데이터는 크게 세 가지이다.

1. state
2. event
3. sideEffect

그렇다면 한 번 고민해보자. 각 데이터를 무엇을 활용해 다루는 것이 적합할까?

1. `state`
- state는 결국 <span style = "background-color:#fff5b1">화면을 나타내는 데이터 상태이다.</span> 따라서 <span style = "background-color:#fff5b1">이전의 상태가 어떻든 현재 상태만이 중요하다.</span>
- 그렇다! 오로지 현재 상태만을 다루는 <span style = "background-color:#fff5b1">StateFlow</span>를 활용할 수 있다.

2. `event`
- 다른 말로 하면 `user-action`이다. user action은 우선 <span style = "background-color:#fff5b1">hot</span>으로 다뤄져야 한다. 또한 여러 명의 구독자가 존재할 수도 있으며, 가능하다면 이전 event를 repeat해야 할 가능성이 존재한다.
- <span style = "background-color:#fff5b1">sharedFlow</span>을 활용할 수 있다.

3. `sideEffect`
- <span style = "background-color:#fff5b1">하나의 구독자만 존재해 한 번만 실행되는 부수효과이다.</span>
- 따라서 <span style = "background-color:#fff5b1">channel</span>을 활용할 수 있다.

## 🔗 마무리하며..

여기까지 coroutine과 함께 공유 데이터를 어떻게 잘 관리할 수 있는지, 또 그 활용 측면에서 `SharedFlow`, `StateFlow`, `Channel`은 각각 어떤 특징이 있는지를 깊이 다뤄보았다. 비슷한 듯 다른 점이 존재하기에 이를 정확히 짚고 넘어가는 것이 중요하며, 쓰임에 알맞게 잘 활용할 수 있어야겠다!

- 참고 문헌
  - <https://elizarov.medium.com/shared-flows-broadcast-channels-899b675e805c>
  - <https://kotlinlang.org/docs/channels.html#pipelines>
  - <https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/>
  - <https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/>