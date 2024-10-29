---
title: "[Compose] full compose 프로젝트에 navigation compose 적용기"
categories:
  - Compose
tag:
  - navigation compose

last_modifeid_at: 2024-10-29
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며

`jetpack navigation`을 활용해 기존에는 navigation을 처리하였다. 하지만 compose migration을 진행하면서 `navigation-compose`를 새롭게 학습해 적용하게 되었다. 전체적인 핵심 개념은 동일하지만, jetpack navigation을 활용할 때보다 작성해야 하는 코드가 더욱 간편해졌다는 것을 느꼈다.

그리고 `Navigation 2.8.0`부터 `navigation compose` 및 `navigation kotlin dsl`에서 type safe를 적용할 수 있게 되었다. 아래 과정은 type safe를 적용한 코드이며 관련해서는 해당 [공식 문서](https://developer.android.com/guide/navigation/design/type-safety)를 참고하길 바란다. 참고로 아래 참고 자료 중 공식 문서 navigation compose codelab은 type safe가 적용되기 이전 버전의 코드이다.

# 🔗 [navigation의 핵심 component](https://developer.android.com/guide/navigation#types)

`jetpack navigation`을 사용하든, `navigation compose`를 사용하든 navigation을 구성하는 핵심 component는 동일하다. 정리해보면 다음과 같다.

![navigation-component](/assets/images/navigation-component.jpeg)

즉, 위의 구성 요소를 어느 navigation library를 사용하느냐와 상관없이 무조건 활용하게 된다는 것이다. 그렇다면 본격적으로 `navigation-compose`를 활용해보자!

# 🔗 navigation-compose 활용

## 1️⃣ 의존성 설정하기

```kotlin
dependencies {
    val nav_version = "2.8.3"

    implementation("androidx.navigation:navigation-compose:$nav_version")
}
```

## 2️⃣ navController 정의

compose에서는 `rememberNavController()`를 통해 navController를 생성할 수 있다.
이때 주의할 점은 <span style = "background-color:#fff5b1">navController를 모든 컴포저블에서 참조할 수 있도록 최대한 상위에 배치</span>해야 한다는 것이다.

```kotlin
val navController = rememberNavController()
```

이렇게 상위에 배치해야 navController를 `single source of truth`로 사용하며 composable을 업데이트할 수 있다. 이는 <span style = "background-color:#fff5b1">[state hoisting](http://developer.android.com/jetpack/compose/state#state-hoisting)</span>하고도 연관된다.

## 3️⃣ Destination 정의

compose에서는 `route`를 정의할 때 `serializable`한 object나 class를 사용한다. 여기서 route는 어떻게 destination으로 이동할지, destination이 어떤 정보를 요구하는지에 대한 정보를 가지고 있다. 이때 Destination을 정의하면 해당 값들을 활용해 `NavHost`에서 `NavGrph`를 생성할 때 사용된다.

```kotlin
@Serializable
sealed class BottomNavDestination(@DrawableRes val iconRes: Int, val route: String) : Destination {
    @Serializable
    data object IssueBoard : BottomNavDestination(R.drawable.baseline_error_outline_24, "이슈")

    @Serializable
    data object Label : BottomNavDestination(R.drawable.outline_issue_label_24, "레이블")

    //...
}
```
필자의 경우, BottomNavDestination을 나타내는 상위 sealed class를 구성해 하위에 여러 destination을 배치하였다.

이때 만약 destination을 `argument`와 함께 정의해야 한다면 `data class`를 활용하고, 그렇지 않다면 `data object`를 활용하자.

## 4️⃣ navHost with navGraph 구성

이제 위에서 정의한 destination을 활용해 navGraph를 구성해보자! 우선 navHost에는 아래와 같이 생성한 navController의 인스턴스와, 시작 destination을 넘겨주어야 한다.

그리고 navHost의 마지막 인자인 `NavGraphBuilder`를 통해 내부에 navGraph를 구성할 destination을 하나씩 정의한다.

```kotlin
NavHost(
    navController = navController,
    startDestination = BottomNavDestination.IssueBoard
) {
    composable<BottomNavDestination.IssueBoard> {
        IssueBoardScreen(
            modifier,
            navigateToIssueDetailScreen = { issueNumber ->
                navController.navigate(NavigateDestination.IssueDetail(issueNumber))
            }
        )
    }
    //...
}
```

그렇다면 `argument를 전달하는 방법`에 대해서도 살펴보자!

```kotlin
@Serializable
data class IssueDetail(
    val issueNumber: Int // 필요한 argument를 선언해준다.
) : NavigateDestination("issue_detail")
```

다음과 같이 우선 필요한 argument를 Destination의 class에 추가해준다.

```kotlin
// 전달
composable<BottomNavDestination.IssueBoard> {
    IssueBoardScreen(
        modifier,
        navigateToIssueDetailScreen = { issueNumber ->
            navController.navigate(NavigateDestination.IssueDetail(issueNumber))
        }
    )
}
// 사용
composable<NavigateDestination.IssueDetail> { backStackEntry ->
    val issueDetail: NavigateDestination.IssueDetail = backStackEntry.toRoute()
    IssueDetailScreen(
        onBackButtonPressed = {
            navController.popBackStack(BottomNavDestination.IssueBoard.route, false)
        },
        issueNumber = issueDetail.issueNumber
    )
}
```

그리고 위와 같이 `backStackEntry.toRoute()`를 호출할 경우, NavigationDestination.Route를 NavBackStackEntry.arguments와 함께 지정한 타입(여기서는 IssueDetail)의 인스턴스로 재생성하게 된다. 이렇게 생성된 인스턴스를 가지고 위에서 추가한 변수에 접근할 수 있게 된다.

```kotlin
/**
 * Returns route as an object of type [T]
 *
 * Extrapolates arguments from [NavBackStackEntry.arguments] and recreates object [T]
 *
 * @param [T] the entry's [NavDestination.route] as a [KClass]
 * @return A new instance of this entry's [NavDestination.route] as an object of type [T]
 */
public inline fun <reified T> NavBackStackEntry.toRoute(): T {
    val bundle = arguments ?: Bundle() 
    val typeMap = destination.arguments.mapValues { it.value.type } // 전달된 argument 가져오기
    return serializer<T>().decodeArguments(bundle, typeMap) // T 객체 argument와 함께 재생성하기
}
```


## 5️⃣ navigate 처리하기

위 코드를 통해서도 쉽게 알 수 있지만 이제 실제 `navigate`를 어떻게 처리하는지에 대해서 다뤄보고자 한다.
위 코드 중 일부를 다시 한 번 가져와 보자.

```kotlin
composable<BottomNavDestination.IssueBoard> {
    IssueBoardScreen(
        modifier,
        // navigate event로 전달
        navigateToIssueDetailScreen = { issueNumber ->
            navController.navigate(NavigateDestination.IssueDetail(issueNumber))
        }
    )
}
```

우선 `navController.navigate(route = "navigateion할 destination type")`을 통해 손쉽게 navigation을 처리할 수 있다.

하지만 위 코드에서 알 수 있듯이 `navController`를 전달해 하위에서 `navigate` 함수를 직접 호출하지 않는다. 위에서 navController를 생성할 때도 말했듯이 `single source of truth`로 navController를 다루기 위해서이다. 또한, compose에서 중요한 `UDF(unidirectional data flow)` 원칙을 지키기 위함이다. 

> 그렇다면 잠깐 `UDF`에 대해 한 번 짚고 넘어가자!
>
> <image src ="/assets/images/unidirection-data-flow.jpeg" width = 400/>
>
> 사진을 통해서도 알 수 있듯이 state는 아래로, event는 위로 향하는 디자인패턴이다. 해당 패턴의 장점은 <span style = "background-color:#fff5b1">state를 display하는 UI와 state를 저장하고 변경하는 부분을 분리할 수 있다는 점</span>이다.
> 
> ## 그렇다면 UDF를 적용했을 때 UI가 업데이트되는 과정을 살펴보자.
> - event: UI에서는 event가 발생했을 때(ex: 버튼 click) 상위로 이벤트를 전달한다.
> - update state: event hanlder에서는 state를 필요에 따라 변경한다.
> - display state: state holder는 state를 하위로 전달하고, UI에서는 해당 state를 화면에 표시하게 된다.
>
> ## 마지막으로 UDF를 적용했을 때의 장점을 확인해보자!
> - `testability`: state와 UI를 분리하기 때문에 두 로직 모두 테스트가 쉬워진다.
> - `state encapsulation`: state의 변경은 one place에서만 진행된다. state 관련 로직이 캡슐화 되기 때문에 일관되지 않는 state로 인한 버그를 최소화할 수 있다.
> - `ui consistency`: observable state holder(ex: stateFlow, liveData)를 활용한다면 업데이트된 state를 바로 UI에 반영할 수 있다.

# 🔗 마무리하며

어떤가? `jetpack navigation`을 활용했을 때와 큰 흐름은 크게 달라지지 않은 것을 확인할 수 있다. 또한, 실제 navigation 처리는 상위 컴포저블에서만 진행하고, 하위 컴포저블에서는 event에 의해서만 navigation을 trigger한다는 점이 `navigation compose`의 가장 큰 장점이지 않을까 싶다. (물론 navController를 직접 전달하게 된다면 이러한 장점을 활용하지 못하게 된다,,) 

만약 full compose로 앱을 구성한다면 `navigation compose`를 활용하자!

# 🔗 참고 자료
- [https://developer.android.com/codelabs/jetpack-compose-navigation#7](https://developer.android.com/codelabs/jetpack-compose-navigation#7)
  - (type-safe가 적용되지 않은 codelab)
- [https://developer.android.com/develop/ui/compose/navigation?hl=ko](https://developer.android.com/develop/ui/compose/navigation?hl=ko)