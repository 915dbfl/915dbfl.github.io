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

> [권장되는 사용 시기]
- ComposeView whether it's the sole element in the View hierarchy, or in the context of a mixed View/Compose screen (not in Fragment).
  - view 계층에서 ComposeView 하나만이 존재하는 경우
  - fragment가 아닌 상태에서 View/Compose가 섞인 화면 속에서 사용될 경우
- ComposeView as an item in a pooling container such as RecyclerView.
  - recyclerView 안 item의 경우

<br>

## 🔗 3. DisposeOnLifecycleDestroyed

- 인자로 전달받은 <span style = "background-color:#fff5b1">lifecycle이 destroy될 때 composition도 해제된다.</span>
- <span style = "background-color:#fff5b1">fragment 환경의 composeView에서 쓰기 용이하다.</span>
    - view는 detached 되었으나 fragment가 destroy되지 않는 경우가 존재하기 때문이다.
- ComposeView가 lifecycle과 1:1 관계일 때 적합하다.

> [권장 사용 시기]
- ComposeView in a Fragment's View.
  - `fragment 안`에서 ComposeView를 사용할 경우

<br>

## 🔗 4. DisposeOnViewTreeLifecycleDestroyed

- view가 attach된 이후 <span style = "background-color:#fff5b1">현재의 window의 viewTreeLifecycleOwner가 destroy될 때 composition이 해제된다.</span>
- composeView를 사용하는 부분에서 <sapn style = "background-color:#fff5b1">lifecycle을 모를 때 사용하면 된다.</span>

> [권장 사용 시기]
- ComposeView in Fragment's View.
  - `fragment 안`에서 ComposeView를 사용할 경우
- ComposeView in a View wherein the Lifecycle is not known yet.
  - lifecycle을 잘 모르는 View 속 ComposeView를 사용할 경우

<br>

## 🔗 5. ComposeView in Fragment 상황에서는 어떤 전략을 사용해야 하지??

- composition는 보통 composeView가 window에서 detached될 때 dispose된다.
    
- fragment에서 ComposeView를 사용할 경우, 해당 전략이 적절하지 않은 이유
  1. <span style = "background-color:#fff5b1">fragment View의 생명주기와 일치해야 한다.</span>
      - compose ui가 state를 저장하기 위해 composition이 `fragment의 view의 lifecycle`을 따라야 한다.
      - 특히, Compose의 Ui 상태를 저장해야 하는 경우, fragment 생명주기와 맞춰져야 한다.
  2. <span style = "background-color:#fff5b1">trasition(화면 전환) 중 상태 유지를 해야 한다.</span>
      - 화면 전환 상태에서 ComposeView가 잠깐 분리되는 순간이 존재한다. 이때 화면에서 분리되었다고 해서 Composition이 해제되면, 중간에 갑자기 UI가 사라지는 문제가 발생할 수 있다.
      - UI는 화면 전환 과정에서도 계속 보여야 한다.

### 🤔 그렇다면 `DisposeOnLifecycleDestroyed`를 사용해도 되지 않을까?? 

Fragment에는 lifecycle이 크게 두 가지가 존재한다.
- fragment 자체의 lifecycle
- fragment의 `view`의 lifecycle

위 두 전략을 비교해본다면 다음과 같다.
- `DisposeOnLifecycleDestroyed`

  fragment 자체 lifecycle이 소멸될 때 composition이 해제된다. 하지만 fragment view와 fragment의 생명주기는 서로 다르다. 따라서 뷰가 파괴된 상태에서도 composition이 해제되지 않아 메모리 누수나 상태 불일치가 발생할 수 있다.

- `DisposeOnViewTreeLifecycleDestroyed`

  fragment의 <span style = "background-color:#fff5b1">view의 생명주기</span>에 맞춰 composition이 관리되기 때문에 fragment 전환이나 애니메이션 도중 UI가 안전하게 유지될 수 있다.


<br>

## 참고
* <https://developer.android.com/jetpack/compose/migrate/interoperability-apis/compose-in-views>
* <https://velog.io/@beokbeok/ViewCompositionStrategy>
* <https://www.youtube.com/watch?v=E4OmRPD332Q>