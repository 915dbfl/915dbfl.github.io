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

(내용 수정: 2025.01.04)

최근 기존 프로젝트에 compose를 도입하면서 `상태관리`의 중요성에 대해 몸소 느끼게 되었다. 현재 프로젝트에서는 MVVM 패턴을 사용하고 있는데 state를 필요한 만큼 선언하고 이를 가져와 composable 함수에서 사용하니 <span style = "background-color:#fff5b1">원하지 않은 결과가 화면에 보이거나 불필요한 recomposition이 발생</span>하는 것을 확인할 수 있었다.

우선적으로 화면당 하나의 state를 만들어 관리할 경우, 일부 요소만 렌더링 되는 문제를 해결할 수 있었다. 하지만 <span style = "background-color:#fff5b1">단순히 하나의 state만을 만들어 관리할 경우, navigation 전환 / snackbar 띄우기와 같은 단일 이벤트와 상태 변경 이벤트를 경우에 따라 직접 구분해 처리해줘야 한다는 불편함이 존재했다.</span>

`MVVM 패턴`에서는 사용자 입력에 따라 어떤 액션을 취할지를 개발자가 직접 판단해 그에 알맞은 presentation logic을 호출해줘야 한다. 하지만 `MVI 패턴`에서는 Intent라는 요소가 사용자 입력에 맞는 액션을 호출함으로써 이벤트에 따라 일관되게 로직을 처리할 수 있다.

compose의 특징 중 하나는 <span style = "background-color:#fff5b1">UDF(단방향 데이터 흐름 - 상태는 아래로만, 이벤트는 위로만 이동한다)이다.</span> 이로 인해 상태는 한 곳에서만 업데이트가 되게 되는데, 이러한 특징은 `MVI 패턴`과 잘 맞는다. 추후 다룰 내용이지만 <span style = "background-color:#fff5b1">MVI 패턴에서는 결국 intent에 의해서만 상태가 변경되기 떄문이다.</span>

이번 게시글에서는 

1. MVI가 정확히 뭔지?
2. compose + MVI를 어떻게 적용할 수 있는지?

를 체계적으로 다뤄보고자 한다!

<br>

## 🔗 MVI의 concept 등장

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

## 🔗 MVI 직접 적용해보며..

우선 MVI 패턴에서는 best practice 코드가 많이 존재하지 않았다. 따라서 참고한 리소스마다 서로 다른 구현 방식을 가지고 있어 어떤 방식이 더 나을지를 스스로 판단해야 했다. 그 중에서도 `orbit`과 같이 별도의 라이브러리를 통해 처리하는 코드도 존재했지만, 필자는 별도의 라이브러리 없이 Channel과 Flow를 활용하여 `State`, `Effect`, `Event`를 직접 구현하는 방식으로 MVI 패턴을 적용해보았다.

결론적으로 기존 MVVM 패턴을 유지하며 event와 effect를 추가적으로 정의한 방식이었는데 그 과정에서 뭔가 새로운 패턴을 적용한다는 느낌보다는 <span style = "background-color:#fff5b1">MVVM에서 이벤트에 따른 상태를 효과적으로 관리하기 위해 추가적인 작업</span>을 진행한다는 느낌이 들었다. <span style = "background-color:#fff5b1">즉, 사용자의 액션을 구분해 Intent로 정의하는 것이 핵심인 것 같다.</span>

(nowinandroid, droidknights 등의 유명한 안드로이드 오픈소스를 살펴본다면 MVI를 적용한 코드를 아직 확인할 수 없다. 화면에서 다루는 하나의 UIState를 구성하고, 단일 이벤트를 처리하는 별도의 변수를 두는 등의 방식으로 상태를 관리하고 있는 것을 확인할 수 있다.)

## 🔗 적용 코드 확인하기 (수정 예정)

[LGTM 프로젝트: SuggestionDetailViewModel 코드](https://github.com/hellokitty-coding-club/LGTM-Android/blob/develop/feature/mission_suggestion/src/main/java/com/lgtm/android/mission_suggestion/ui/detail/SuggestionDetailViewModel.kt)를 본다면 우선 viewModel에서 Input, Output interface를 구현하고 있는 것을 볼 수 있다.

또한, `State`, `Effect`, `Event` 각각에 대해서 왜 해당 구현 방식을 적용하게 되었는지에 대한 내용을 [다음 게시글: [Flow / Channel] Flow, Channel 쓰임새를 알고 잘 활용하기](https://915dbfl.github.io/android/flow_and_channel/)을 통해 확인할 수 있다.

여기서 `Input`은 <span style = "background-color:#fff5b1">사용자 액션을 정의하는 intent에 해당한다.</span> 그리고 outPut은 이러한 사용자 액션으로 발생한 결과(state, uiEffect)를 가지고 있는 interface이다.

사용자 입력이 발생하면 그에 맞는 사용자 액션 정의 intent를 직접 호출한다. 그러면 그에 맞게 로직이 실행되고, 그에 따라 state가 업데이트 되고, effect가 발생하게 된다. 

> 필자의 방식은 MVI 패턴의 간단한 구현 방식 중 하나이다. MVI 패턴을 검색하면 <span style = "background-color:#fff5b1">Reducer(발생한 event를 통해 새로운 상태 갱신)라는 개념</span>이 많이 보이는데 해당 개념이 적용되는 대신 액션에 따른 특정 event를 직접 호출함으로써 reducer의 로직을 event에 따라 분리하였다. 이로 인해 상태 변경 로직이 여러 군데에 존재해 유지 보수의 어려움이 존재한다.

> 대부분의 MVI 패턴 예시 코드를 확인해본다면 `Reducer`가 적용되어 있다. 그 이유는 상태 변경 로직을 한곳으로 둠으로써 유지보수 및 테스트를 용이하게 하기 위함이다. 관련해서 조금 더 학습한 후, 코드 개선을 진행해봐야겠다.


## 🔗 참고자료
- <https://proandroiddev.com/mvi-a-new-member-of-the-mv-band-6f7f0d23bc8a>
- <https://medium.com/swlh/mvi-architecture-with-android-fcde123e3c4a>
- <https://www.charlezz.com/?p=46365>
- <https://jyotibhambhu.medium.com/implementing-mvi-with-reducers-in-android-a-step-by-step-tutorial-4f23ca426b2f>