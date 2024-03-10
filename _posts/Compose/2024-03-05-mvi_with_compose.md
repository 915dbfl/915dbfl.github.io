---
title: "[Compose] MVI with compose"
excerpt: "MVI에 대해 DeepDive"
categories:
  - Compose
tag:
  - MVI
  - compose
  - MVI deepdive

last_modifeid_at: 2024-03-05
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며
최근 기존 프로젝트에 compose를 도입하면서 `상태관리`의 중요성에 대해 몸소 느끼게 되었다. 현재 프로젝트에서는 MVVM 패턴을 사용하고 있는데 state를 필요한 만큼 선언하고 이를 가져와 composable 함수에서 사용하니 <span style = "background-color:#fff5b1">원하지 않은 결과가 화면에 보이거나 불필요한 recomposition이 발생</span>하는 것을 확인할 수 있었다. 

```kotlin
// 현재 코드

@Composable
fun SuggestionDetailScreen(
    viewModel: SuggestionDetailViewModel = hiltViewModel(),
    onBackButtonClick: () -> Unit
) {
    val suggestionDetailState by viewModel.detailState.collectAsStateWithLifecycle()
    val likeSuggestionState by viewModel.likeSuggestionState.collectAsStateWithLifecycle()
    ...
}
```

화면이 복잡할수록 위와 같이 composable에서 접근하는 state가 많아지게 되는데, 이럴 경우에는 어떻게 상태를 관리해야 불필요한 리소스 낭비를 방지할 수 있을까 고민하게 되었다.

안드로이드 개발을 하면서 자주 챙겨보는 `nowinandroid`의 코드를 이것저것 확인해보던 중, 다음과 같이 feature마다 uiState를 정의해 활용하는 것을 볼 수 있었다.

```kotlin
sealed interface OnboardingUiState {
    /**
     * The onboarding state is loading.
     */
    data object Loading : OnboardingUiState

    /**
     * The onboarding state was unable to load.
     */
    data object LoadFailed : OnboardingUiState

    /**
     * There is no onboarding state.
     */
    data object NotShown : OnboardingUiState

    /**
     * There is a onboarding state, with the given lists of topics.
     */
    data class Shown(
        val topics: List<FollowableTopic>,
    ) : OnboardingUiState {
        /**
         * True if the onboarding can be dismissed.
         */
        val isDismissable: Boolean get() = topics.any { it.isFollowed }
    }
}
```

해당 부분은 바로 `MVI` 아키텍처가 적용된 부분이다! <span style = "background-color:#fff5b1">compose + MVI</span>을 활용하면 상태를 효과적으로 관리할 수 있다!

그래서 이번 게시글에서는 

1. MVI가 정확히 뭔지?
2. compose + MVI를 어떻게 적용할 수 있는지?

를 체계적으로 다뤄보고자 한다!

<br>

## 🔗 MVI의 concept 등장

~~(마크 다운에서 갑자기 구글 드라이브 이미지가 안된다..ㅠㅠ 찾아보니 다른 사람도 같은 이슈를 겪고 있는 것 같아..흑흑 조만간 블로그를 옮기든 해결책을 찾아봐야겠당..)~~

```
     안녕, B?
   --------->
A             B
  <----------
    안녕, A?
  
```
위의 상황을 확인해보자! A와 B는 아무 방해를 받지 않고 대화를 하고 있다. 한 사람은 듣고 다른 한 사람은 들은 것에 대해 반응을 한다.

만약 위 상황에서 B를 컴퓨터라고 생각해보자! 그렇다면 위의 흐름은 어떻게 바뀔까??

```
     -------   B💻 ---------
     |                     |
     ↓                     ⎮
output🖥️                 input🖱️
input👁️                  output🫳
     ⎮                     ↑
     |                     |
     --------  A👤 ---------
```

사람 대신 컴퓨터가 생겼기 때문에 컴퓨터 입장에서는 `input`이 들어올 키보드와 마우스 같은 도구가 필요할 것이고 `output`이 나갈 출력장치 또한 필요할 것이다.

일반적으로 input과 ouput을 연결하기 위해 우리는 수학적으로 `function`을 활용한다. 따라서 우리는 각 input, ouput을 아래와 같이 function으로 표기할 수 있다.

```
-> 이것이 바로 MVI의 base graph이다.

     -------   B💻 ---------
     |       model()       |
     ↓                     ⎮
view()🖥️                 intent()🖱️
     ⎮                     ↑
     |       user()        |
     --------  A👤 ---------
```
user()의 결과는 intent의 input으로 넘어가고, intent의 결과는 model()의 input으로 들어간다...(반복)

따라서 위 흐름에서 우리가 알고 넘어가야 할 부분은 다음과 같다.

1. <span style = "background-color:#fff5b1">ouput은 다음 function의 input이 된다.</span>
2. <span style = "background-color:#fff5b1">data는 하나의 방향으로 흘러간다.(unidirection)</span>

> intent(user(view(model(intent(user())))))

위의 circular를 표현하면 다음과 같이 표현된다. 이때 user는 우리 자신이므로 제외를 시켜보자!

> view(model(intent()))

위의 pure function을 보자!

1. <span style = "background-color:#fff5b1">output은 오로지 들어오는 input에 의해서만 결정된다.</span>(side effect와 상관없이!)

2. input이 같다면 output도 동일하다.

여기까지 진행했다면 우리는 자연스럽게 MVI의 핵심 개념인 `view`, `model`, `intent`를 다루게 된다!

<br>

## 🔗 MVI DeepDive 전,, Side Effect 다루기!

보통 Api을 호출하거나 DB 작업을 하면 그 결과값이 model에 입력값으로 처리된다. 이때 model에 입력값으로 처리가 되면서 동시에 <span style = "background-color:#fff5b1">다른 구성 요소의 side effect가 실행될수도 있다.</span> 이러한 사이드 이펙트의 결과는 <span style = "background-color:#fff5b1">아무 것도 아니거나 새로운 intent가 될수 있다.</span>

예를 들어 Toast를 띄우거나, Logging을 쏘거나 하는 작업들은 이벤트이지만 그 결과가 없기에 상태를 변경할 필요가 없다. 이런 작업들을 sdieEffect로 따로 두어 처리를 하는 것이다.

`Side Effect`가 포함된 graph를 살펴보자!

```
                ⌈---- side effects ←----
                ↓                      |
     ------------ B💻 -------------    |
     |            model()         |    |
     ↓                            ⎮    |
view()🖥️                        intent()🖱️
     ⎮                            ↑
     |           user()           |
     ------------ A👤 ------------
```

- intent()의 결과는 model()의 input으로 넘어간다.
- 동일한 시간에, intent()는 side effect를 유발한다.
- side effect의 결과는 아무 것도 아니거나, 새로운 intent()이다.
    - model의 input으로 들어가거나
    - 또 다른 side effect를 run하거나

그렇다면 이제 MVI에 대해 본격적으로 deepdive하는 시간을 가져보자!

<br>

# 🔗 MVI DeepDive

## 🔗 MVI 핵심 개념 다루기

> 1) <span style = "background-color:#fff5b1">Intent</span>

여기서 말하는 intent는 우리가 navigate를 처리할 때 사용하는 intent와는 다른 개념이다. 

- <span style = "background-color:#fff5b1">intention to change the state</span>를 의미한다. 즉, 상태를 바꾸는 것을 의미한다.
- 모든 UI의 변경은 `intent()`의 결과이다.

> 2) <span style = "background-color:#fff5b1">Model</span>

여기서 말하는 model 또한 우리가 지금까지 MVVM을 활용할 때 사용했던 model과는 다른 개념이다. MVVM에서는 하나의 data holder로 model을 만들어 사용했을 것이다. 

- model은 <span style = "background-color:#fff5b1">UI에 반영될 "immutable" state</span>를 의미한다.

> 3) <span style = "background-color:#fff5b1">View</span>

- model의 값을 view에 나타내는 역할
- UI 그 자체라 이해하자.

## 🔗 State 추가 설명!

- 어느 순간이든, <span style = "background-color:#fff5b1">"하나의 state"만</span>이 존재한다.
- <span style = "background-color:#fff5b1">intent()에 의해 새로운 상태를 만드는 것만</span>이 state를 바꾸는 유일한 방법이다.

## 🔗 MVI 장단점 정리

> 1) 장점

- <span style = "background-color:#fff5b1">상태 충돌이 없다.</span>
    - 모든 순간, 하나의 state만이 존재
- <span style = "background-color:#fff5b1">단방향 데이터 흐름</span>
    - 로직이 예측 가능하고 문제 추적이 쉽다.
- <span style = "background-color:#fff5b1">immutability로 인한 장점</span>
    - 각 과정의 output들이 immutable하기 때문에 thread safety와 공유 가능의 이점을 가진다.
- debuggability
    - unidirectional data flow의 이점으로 디버깅이 쉬워진다.
- testability

> 2) 단점

- boiler plate 코드 증가
- 구현의 복잡성
- <span style = "background-color:#fff5b1">작은 변경에도 intent를 통한 사이클이 필요하다.</span>

## 🔗 참고자료
- <https://proandroiddev.com/mvi-a-new-member-of-the-mv-band-6f7f0d23bc8a>
- <https://medium.com/swlh/mvi-architecture-with-android-fcde123e3c4a>
- <https://www.charlezz.com/?p=46365>