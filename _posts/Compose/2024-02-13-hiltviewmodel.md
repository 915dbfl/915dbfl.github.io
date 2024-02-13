---
title: "[Compose] viewModel() vs hiltViewModel()"
excerpt: "viewModel 적용 scope의 차이!"
categories:
  - Compose
tag:
  - hiltviewmodel

last_modifeid_at: 2024-02-13
toc: true
toc_sticky: true
search: true
---

## 들어가며: viewModels()

`viewModels()`

우리는 기존 안드로이드 프로젝트 각 화면에서 `viewModels`를 이용해 손쉽게 viewModel을 생성할 수 있었다. 이때 생성된 viewModel은 생성한 컴포넌트(activity, fragment..)의 생명주기를 따랐었다.

그렇다면 compose에서도 이렇게 viewModel를 생성할 수 있는 간단한 방법이 존재할까?? 

## viewModel()

```kotlin
class MyViewModel : ViewModel() { /*...*/ }

// import androidx.lifecycle.viewmodel.compose.viewModel
@Composable
fun MyScreen(
    viewModel: MyViewModel = viewModel()
) {
    // use viewModel here
}
```

기존과 마찬가지로 `viewModel()`을 통해서 손쉽게 composable함수에서 viewModel을 생성할 수 있다. 

⛔️ 이때 `lifecycle`과 관련해 주의할 점이 있다!
> 1. screen-level composable 함수에서 viewModel()을 사용하자!
 
> 2. 이렇게 생성한 viewModel을 하위 composable 함수에 pass down하지 말자.

## hiltViewmodel()은 뭔데??

공식 문서를 보면 `hiltViewModel()`이라는 것도 존재한다.

그렇다면 viewModel()과 차이점이 뭘까?? 우선 scope에 존재하는 viewModel을 반환하거나 해당하는 viewModel이 없을 경우, 새로운 viewModel을 생성하는 점에서는 동일하다.

그렇다면 이제부터 차이점을 알아보자!

## viewModel / hiltViewModel 내부 코드 확인!!

| 1. 우선 viewModel부터!

```kotlin
@Suppress("MissingJvmstatic")
@Composable
public inline fun <reified VM : ViewModel> viewModel(
    viewModelStoreOwner: ViewModelStoreOwner = checkNotNull(LocalViewModelStoreOwner.current) {
        "No ViewModelStoreOwner was provided via LocalViewModelStoreOwner"
    },
    key: String? = null,
    factory: ViewModelProvider.Factory? = null,
    extras: CreationExtras = if (viewModelStoreOwner is HasDefaultViewModelProviderFactory) {
        viewModelStoreOwner.defaultViewModelCreationExtras
    } else {
        CreationExtras.Empty
    }
): VM = viewModel(VM::class.java, viewModelStoreOwner, key, factory, extras)
```

여기서 viewModelStoreOwner값을 살펴보자! viewModel에서는 기본값으로 `LocalViewModelStoreOwner.current`를 사용한다. 해당 값을 쭉쭉 따라가다 보면 결국 `최상위 View를 owner로 지정하는 것`을 알 수 있다.

| 2. hiltViewModel의 경우는?

```kotlin
@Composable
inline fun <reified VM : ViewModel> hiltViewModel(
    viewModelStoreOwner: ViewModelStoreOwner = checkNotNull(LocalViewModelStoreOwner.current) {
        "No ViewModelStoreOwner was provided via LocalViewModelStoreOwner"
    },
    key: String? = null
): VM {
    val factory = createHiltViewModelFactory(viewModelStoreOwner)
    return viewModel(viewModelStoreOwner, key, factory = factory)
}

@Composable
@PublishedApi
internal fun createHiltViewModelFactory(
    viewModelStoreOwner: ViewModelStoreOwner
): ViewModelProvider.Factory? = if (viewModelStoreOwner is HasDefaultViewModelProviderFactory) {
    HiltViewModelFactory(
        context = LocalContext.current,
        delegateFactory = viewModelStoreOwner.defaultViewModelProviderFactory
    )
} else {
    // Use the default factory provided by the ViewModelStoreOwner
    // and assume it is an @AndroidEntryPoint annotated fragment or activity
    null
}
```

hiltViewModel에서도 viewModel을 가져다 사용하고 있다. 여기서 주목할 점은 viewModel을 생성할 때 전해주는 factory이다.

createHiltViewModelFactory를 살펴보자! 여기서 `viewModelStoreOwner`가 `HasDefaultViewModelProviderFactory`를 구현했느냐에 따라서 factory값이 달라지는 것을 알 수 있다. 이때 HasDefaultViewModelProviderFactory를 구현하는 component로는 activity, fragment 등이 있다. 

결국, HasDefaultViewModelProviderFactory를 구현하는 경우 `해당하는 component의 스코프에 따라` viewModel이 생성되게 되고, 그게 아니라면 기존 viewModel()과 동일하게 동작하는 것을 알 수 있다.

## 그래서 결론이 뭔데??

그렇다면 우리가 내부 코드를 통해 알아낸 것을 요약하면 다음 한 줄이다. 

| viewModel()은 최상위 view의 scope를 따라 vieweModel이 생성된다.

| hiltViewModel()은 최상위 view가 아닌 hiltviewModel을 호출한 component scope에 따라 viewModel()을 생성된다.

## 📝 참고 자료
* <https://developer.android.com/jetpack/compose/custom-modifiers>
* <https://medium.com/kenneth-android/compose-hiltviewmodel-%EA%B3%BC-viewmodel-%EC%B0%A8%EC%9D%B4-6d5412efcb19>