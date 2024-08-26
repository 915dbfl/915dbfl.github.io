---
title: "[Navigation] Navigation backStack 자세히 파헤쳐보기"
excerpt: "bottomNavigationBar로 진행되는 naviagation에서는 backStack이 어떻게 관리되고, 처리될까?"
categories:
  - Android
tag:
  - Android
  - Navigation
  - Navigation Backstack

last_modified_at: 2024-08-26
toc: true
toc_sticky: true
search: true
---


## 배경) A, B Fragment만 존재하는 ButtonNavigationBar / C Fragment 버튼을 통해 C Fragment로 이동!

<img width="700" alt="스크린샷 2024-08-25 오후 9 36 44" src="https://gist.github.com/user-attachments/assets/10362530-6f13-410b-afd7-79b04cc6a0d3">


Jetpack Navigation을 적용한 상태에서 다음과 같이 A, B, C fragment가 존재하는 상태이다.

다음과 같이 BottomNavigationBar에는 A, B, C 각각의 버튼이 존재하고, B에서 `C Fragment` 버튼을 누르면 내부적으로 Navigation Controller를 활용해 C Fragment로 이동이 처리된다.

![상황](https://gist.github.com/user-attachments/assets/123693cc-8f26-44ed-9259-7ccd5d10e759)


## 상황) A → B → C로 갔다, B를 눌렀는데 C가 보인다?

> 🫥 A → B → `C Fragment`(화면 버튼) 버튼 클릭했을 때, `B Fragment`(바텀)을 클릭했을 경우
> 
> 
> `예상 화면`
> 
> B Fragment
> 
> `결과 화면`
> 
> C Fragment
> 

지금 보면 `B Fragment`를 클릭해도 화면에 `C Fragment`가 표시되는 것을 확인할 수 있다. 결국, B Fragment를 눌러도 C Fragment를 눌러도 C Fragment가 보이는 상황이 된다.

![결과](https://gist.github.com/user-attachments/assets/460aeced-7a3a-45a6-be60-6fb287d48a54)

## 확인1) Fragment Lifecycle Callback 확인하기

앱을 실행하고 `A → B → C Fragment Button`을 누른 상황까지 우선 빠르게 log를 확인해보자!

(지금부터 ButtonNavigationBar의 각 icon은 A, B, C라 부르고, B Fragment 화면상 `C Fragment` 버튼은 C버튼이라고 칭하겠다)

### 1️⃣ 앱 실행 → A가 눌러진 상황

<img width="1233" alt="스크린샷 2024-08-25 오후 10 03 40" src="https://gist.github.com/user-attachments/assets/d189c646-c719-444e-9a30-4460c1126dfa">

lifecycle callback log를 통해 확인할 수 있듯이 Activity와 AFragment가 생성되고 화면에 보여지는 것을 확인할 수 있다.

- Activity
    - `onCreate -> onStart -> onResume`
- AFragment
    - `onAttached -> onCreate -> onViewCreated -> onStart -> onResume`

### 2️⃣ B → C버튼을 누를 경우

<img width="700" alt="스크린샷 2024-08-26 오후 4 09 14" src="https://gist.github.com/user-attachments/assets/b3c755d3-9758-42e2-8854-c3ac567ba168">

A화면에서 B화면으로 전환될 경우의 AFragment와 BFragment의 lifecycle은 다음과 같이 변화한다.

- AFragment
    - `onPause -> onStop`
- BFragment
    - `onAttached -> onCreate -> onViewCreated -> onStart`
- AFragment
    - `onViewDestroy`
- BFragment
    - `onResume`

해당 과정이 `B -> C버튼`을 눌렀을 때도 동일하게 나타난다.

<img width="701" alt="스크린샷 2024-08-26 오후 4 09 49" src="https://gist.github.com/user-attachments/assets/6505de69-925e-4123-bca7-594c76edf4d6">

### 3️⃣ C를 누를 경우

여기서 C를 누를 경우, A → B를 눌렀을 때와는 살짝 다른 lifecycle callback을 확인할 수 있다.

<img width="850" alt="스크린샷 2024-08-25 오후 10 17 49" src="https://gist.github.com/user-attachments/assets/55696664-cd2b-47c0-aef4-7e4091f26816">

- BFragment
    - `saveInstanceState -> onDestroy -> onDetached`
    - BFragment가 소멸하는 것이니 여기까지는 예상했던 결과이다.
- 2번 CFragment
    - C를 누르면서 생긴 CFragment이므로 예상한 결과이다.

> 🫥 의문1) 여기서 소멸되는 C Fragment는 뭘까?
> 
> - 생성되는 CFragment, 소멸되는 CFragment는 서로 같은 것일까?

그렇다면 의문을 바로 해결하기 전에 다시 한 번 B를 눌러 lifecycle callback을 확인해보자!

### 4️⃣ B를 다시 누를 경우

<img width="814" alt="스크린샷 2024-08-25 오후 10 23 13" src="https://gist.github.com/user-attachments/assets/ef463c68-4e97-4f37-b208-f1cd4e873615">

lifecycle callback을 본다면 또 다시 2개의 CFragment에 대한 lifecycle callback이 찍히는 것을 확인할 수 있다.

- 2번 CFragment
    - B를 누름으로써 화면에 보여졌었던 CFragment가 소멸되는 과정이므로 예상한 결과이다.
- BFragment
    - B를 누르면서 B화면이 보여져야 하므로 B가 생성되는 과정, 예상한 결과이다.

> 🫥 의문2) 여기서 생성되는 CFragment는 뭘까?
> 
> - 또한, 화면에서 B가 아닌 CFragment가 보인다!

### lifecycle을 통해 예측할 수 있는 것

`B → C버튼을 통해 생성한 CFragment`와 `C를 통해 생성한 CFragment`는 서로 다른 Fragment이다.

## 확인2) findNavController를 통해 A, B, C item에 대한 backStack을 확인해보자!

위의 상황(A → B → C버튼)을 진행한 상태에서 각 A, B, C에 대한 backStack을 확인해보자!

확인 방법은 간단하다. A, B, C Fragment의 `onViewCreated`에서 `navController를 가져와 backStack을 찍어볼 것이다.`

```kotlin
    @SuppressLint("RestrictedApi")
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
				...
        Log.i("FragmentBackStack", this.javaClass.simpleName + ": \n" + findNavController().currentBackStack.value.joinToString("\n") { it.destination.toString() })
    }
```

### 1️⃣ AFragment

<img width="1223" alt="스크린샷 2024-08-25 오후 10 45 25" src="https://gist.github.com/user-attachments/assets/ab756472-4d2f-4299-948f-91414238e598">

- backStack
    - AFragment

### 2️⃣ BFragment

<img width="1225" alt="스크린샷 2024-08-25 오후 10 45 12" src="https://gist.github.com/user-attachments/assets/731c6402-f351-49dc-baf5-ed147be12e43">

- backStack
    - AFragment
    - BFragment
    - CFragment

> 👀 `B를 눌렀을 때 왜 CFragment가 화면에 출력되었는지 backStack을 통해 알 수 있었다!`
> 
> - b를 눌렀을 때 navController가 가지는 backStack에는 A→B→C가 존재하게 되며, stack 가장 상단에 있는 CFragment가 화면에 보이게 된다.

### 3️⃣ CFragment

<img width="1227" alt="스크린샷 2024-08-25 오후 10 46 43" src="https://gist.github.com/user-attachments/assets/454c8a5d-7fc0-45ac-81c8-a89c46a461a4">

- backStack
    - AFragment
    - CFragment

### backStack을 통해 알 수 있는 것

1. `BottomNaviagion는 item별로 backStack이 관리된다.`
2. `stack의 시작은 startDestination이다.`
    - 여기서 startDestination으로 AFragment가 설정되어 있다.
    - 모든 아이템에 대한 backStack의 시작이 `AFragment` 인 것을 확인할 수 있다.

## 확인3) A→B→C버튼 속 CFragment ≠ A→C 속 CFragment

이제 마지막으로 `A->B->C버튼`을 진행했을 때 존재하는 CFragment와 `A->C`의 CFragment가 서로 다른 Fragment instance인지를 확인할 것이다.

위에서 사용한 backStackEntry에서 제공하는 멤버변수로 구분할 수 있는 방법이 있을까 알아보았지만, 아쉽게 방식을 찾지 못했다.

필자는 Fragment에 전달되는 argument로 비교해보고자 한다!

```kotlin
    <fragment
        android:id="@+id/CFragment"
        android:name="kr.co.fastcampus.part4plus.baseracegame.feature.game.CFragment"
        android:label="C Fragment"
        tools:layout="@layout/fragment_c">
        <argument
            android:name="url"
            app:argType="string"
            android:defaultValue="https://www.google.com" />
    </fragment>
```

```kotlin
        binding.btnCFragment.setOnClickListener {
            val url = "https://m.naver.com"
            findNavController().navigate(
                BFragmentDirections.actionBfragnetToCfragment(
                    url
                )
            )
        }
```

- navigation action의 `url argument`
    - default = `https://google.com`
    - B→C버튼 = `https://naver.com`

그리고 이를 CFragment에서 webView로 페이지를 한 번 띄워보았다.

![웹뷰 결과](https://gist.github.com/user-attachments/assets/ae7bca41-8db7-42ca-8732-d840e73d953f)

결국 두 개의 cFragment는 서로 다른 fragment instance라는 것도 알 수 있다!

## backStack을 관리해 아래의 상황을 만들어보자!

여기까지 lifecycle callback과 backStack logging을 통해 왜 `A -> B -> C버튼` 의 경우 B를 눌렀을 때 BFragment가 아닌 CFragment가 띄워지는지 알 수 있었다.

BottomNavigationBar의 item별로 관리되는 backStack을 적절히 수정하여 다음의 상황을 만들어보자!

> 요구사항
> 
> - A → B → C버튼을 누른 후, B를 눌렀을 때 BFragment가 화면에 보이도록 한다.
> - A → B → C버튼을 누른 후, back버튼을 눌렀을 때 BFragment가 보이도록 한다.

### 1️⃣ [popUpTo 활용하기](https://developer.android.com/guide/navigation/backstack?hl=ko)

navigation action을 정의할 때 `popUpTo` 속성을 설정할 수 있다.

```kotlin
    <fragment
        android:id="@+id/BFragment"
        android:name="kr.co.fastcampus.part4plus.baseracegame.feature.game.BFragment"
        android:label="B Fragment"
        tools:layout="@layout/fragment_b">
        <action
            android:id="@+id/action_bfragnet_to_cfragment"
            app:destination="@id/CFragment"
            app:popUpTo="@id/BFragment" // popUpTo 설정
            app:popUpToInclusive="false"
            app:enterAnim="@anim/nav_default_enter_anim"
            app:exitAnim="@anim/nav_default_exit_anim"
            app:popEnterAnim="@anim/nav_default_pop_enter_anim"
            app:popExitAnim="@anim/nav_default_pop_exit_anim" />
    </fragment>
```

- `popUpTo` : 스택에서 해당 destination 위의 항목을 모두 팝한다.
- `popUpToInclusive` : 지정한 대상으로 이동한 후에 이를 백 스택에서 팝할지 결정

필자는 위의 코드가 다음과 같이 동작하기 때문에 원하는 결과를 얻을 수 있다 생각했다.

- `popUpTo = BFragment`로 BFragment 스택 이후 모두 제거
- `inclusive = falsea` 로 BFragment는 유지

⇒ `A-> B`만 스택에 존재! 따라서 back을 누르면 B가 출력되고, 다시 bottomNavigationBar의 b를 누르면 b가 출력된다!! **^^** (잘못된 이해)

### 2️⃣ popUpTo 제대로 이해하기) ✨navigate 전에 popUp이 일어난다!✨

> 잘못된 이해) `popUpTo = B` 로 설정하고 `inclusive = false`로 하면 stack에 A→B만 존재해야 하는 거 아닌가?
> 

여기서 우리는 `popUpTo`가 언제 어떻게 실행되는지를 다시 한 번 짚고 넘어가야 한다.

- popUpTo는 스택의 최상단부터 가장 마지막 하위 target 이후까지 모두 pop을 진행한다.
- ✨popUpTo는 navigating 전에 일어난다.✨

> 따라서 `CFragment가 화면에 추가되기 전`에 스택을 탐색한다.
> 
> - B 이후의 스택을 모두 제거하고 B는 `inclusive = false`이므로 스택에 남겨둔다.
> - navigate를 통해 C가 스택에 추가된다.
> 
> ⇒ 따라서 결과적으로 스택이 `A -> B -> C` 이 되는 것이다.
> 

### 3️⃣ bottomNavigationBar의 item 대상이 없다면 스택이 복원되지 않는다.

위와 같은 코드에서 `inclusive = true`만 수정해 다시 백스택을 확인해보았다. 여기서 또 새로운 점을 확인할 수 있었는데,,,

- `A -> B -> C버튼`을 눌렀을 때
    
    <img width="1201" alt="스크린샷 2024-08-26 오전 1 06 30" src="https://gist.github.com/user-attachments/assets/32a2180f-d86a-43b3-9ac0-353e55e13a79">
        
    - popUpTo와 inclusive로 인해 BFragment가 백스택에서 사라진 것을 확인할 수 있다.

> 여기서 질문! 만약 이 상태에서 BottomNavigationBar의 b를 누르면 어떻게 될까?
> 
> - 필자는 stack은 차곡차곡 쌓이는 거니까 `A -> C -> B` 형태의 결과를 예상하였다.

- `BottomNavigation B를 다시 눌렀을 경우`
    
    <img width="1191" alt="스크린샷 2024-08-26 오전 1 09 25" src="https://gist.github.com/user-attachments/assets/4a75ce5d-c7fb-4b1a-afdd-6807472cda4a">
    
    - C에 대한 스택은 온데간데 사라지고, `A -> B`로 스택이 초기화된 것을 확인할 수 있다.

> 사실 왜 이런지에 대한 정확한 이유를 찾지는 못했지만, GPT에게 물어본 결과, 다음의 답을 추론할 수 있었다.
> 
> - B가 완전히 제거된 상태에서 B를 누를 경우, `B가 다시 나타나면서 스택은 [a -> b]로 초기화가 된다.`

### 4️⃣ 결론) 해결 방법 고민

위의 내용들을 토대로 이제 해결방법을 다시 한 번 고민해보자!

- A → B → C버튼을 누른 후, B를 눌렀을 때 BFragment가 화면에 보이도록 한다.
    
    ⇒ navigation을 진행한 후에 스택에서 C를 팝하거나, B를 pop하는 과정이 필요하다.
    
- A → B → C버튼을 누른 후, back버튼을 눌렀을 때 BFragment가 보이도록 한다.
    - navigation이 일어나기 전 `A -> B` 의 스택들 중 삭제해야 할 요소는 없다.
        - popUpTo는 navigation이 일어나기 전에 실행되기 때문에 `popUpTo = B`를 사용할 수 없다.
    - `popUpTo = C`도 사용할 수 없다.
        - navigation이 일어나기 전에 백스택에 C가 존재하지 않기 때문에 무의미하다.

필자의 경우 `setOnItemSelectedListener`를 새롭게 정의해서 해결했다.

```kotlin
binding.btnBottomNavBar.setOnItemSelectedListener { item ->
    navController.popBackStack(item.itemId, true)
    navController.navigate(item.itemId)
    true
}
```

- 우선 `destination` 위에 쌓인 대상을 모두 pop을 한다. 이때 destination까지 pop을 진행한다.
- 해당 destination으로 navigate를 진행해서 화면 전환을 진행한다.