---
title: "[Compose] 언제 composition을 제거해야 할까?"
excerpt: "viewCompositionStrategy 종류와 사용법 알기!"
categories:
  - Compose
tag:
  - viewCompositionStrategy
  - DisposeOnDetachedFromWindow
  - DisposeOnDetachedFromWindowOrReleasedFromPool
  - DisposeOnLifecycleDestroyed
  - DisposeOnViewTreeLifecycleDestroyed

last_modifeid_at: 2024-02-26
toc: true
toc_sticky: true
search: true
---

## 🔗 들어가며
<span style = "background-color:#fff5b1">fragment 화면을 compose로 구성
</span> 또는 <span style = "background-color:#fff5b1">ComposeView</span>을 활용할 setContent 안에 해당 코드를 넣은 기억이 있을 것이다.

```kotlin
    setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed)
```

그렇다면 이 코드의 역할은 무엇일까? 말 그래도 <sapn style = "background-color:#fff5b1">언제 composition을 제거할지에 대한 전략을 정의</span>하는 것이다. 이를 잘 지정해주어야 메모리 릭이나 state 손실과 같은 이슈를 방지할 수 있다.

<br>

## 🔗 ViewCompositionStrategy 종류

- `DisposeOnDetachedFromWindow`
- `DisposeOnDetachedFromWindowOrReleasedFromPool (default)`
- `DisposeOnLifecycleDestroyed`
- `DisposeOnViewTreeLifecycleDestroyed`

우선 다음의 4가지 전략이 있다. 그렇다면 각 전략을 언제 활용해야 하는지에 대해 자세히 살펴보자!

<br>

## 🔗 1. DisposeOnDetachedFromWindow

- activity는 window에 attach될 때 그려진다.
- <span style = "background-color:#fff5b1">compose를 activity에서 활용하거나 `ViewGroup.removeView..`가 진행될 때 트리거 된다.</span>
- 즉, <span style = "background-color:#fff5b1">window에서 detached될 때 composition을 제거한다.</span>
- 현재, DisposeOnDetachedFromWindowOrReleasedFromPool로 대체되었다.

<br>

## 🔗 2. DisposeOnDetachedFromWindowOrReleasedFromPool (default)

- compose가 <span style = "background-color:#fff5b1">recyclerView과 같이 pooling container 안에서 존재할 때</span>
- recyclerview 안에 compose가 존재할 때 DisposeOnDetachedFromWindow를 사용한다면 스크롤 중 각 아이템에 대해 dispose + recreate가 빈번하게 발생해 성능 이슈가 발생할 수 있다!
    - [Jetpack Compose Interop: Using Compose in a RecyclerView](https://medium.com/androiddevelopers/jetpack-compose-interop-using-compose-in-a-recyclerview-569c7ec7a583)
- 해당 전략을 사용할 경우, <span style = "background-color:#fff5b1">pooling container 자체가 창에서 분리되거나 삭제될 때</span> composition이 제거된다.

<br>

## 🔗 3. DisposeOnLifecycleDestroyed

- 인자로 전달받은 <span style = "background-color:#fff5b1">lifecycle이 destroy될 때 composition도 해제된다.</span>
- <span style = "background-color:#fff5b1">fragment 환경의 composeView에서 쓰기 용이하다.</span>
    - view는 detached 되었으나 fragment가 destroy되지 않는 경우가 존재하기 때문이다.
- ComposeView가 lifecycle과 1:1 관계일 때 적합하다.

<br>

## 🔗 4. DisposeOnViewTreeLifecycleDestroyed

- view가 attach된 이후 <span style = "background-color:#fff5b1">현재의 window의 viewTreeLifecycleOwner가 destroy될 때 composition이 해제된다.</span>
- composeView를 사용하는 부분에서 <sapn style = "background-color:#fff5b1">lifecycle을 모를 때 사용하면 된다.</span>

- 🔗 해당 전략을 fragment에서도 사용하는 이유?
    - fragment가 activity lifecycle을 따를 수도 있고 fragment lifecycle을 따를 수도 있기 때문이다. 이렇게 lifecycle을 모를 경우 해당 전략을 사용하면 된다!

<br>

## 참고
* <https://developer.android.com/jetpack/compose/migrate/interoperability-apis/compose-in-views>
* <https://velog.io/@beokbeok/ViewCompositionStrategy>
* <https://www.youtube.com/watch?v=E4OmRPD332Q>