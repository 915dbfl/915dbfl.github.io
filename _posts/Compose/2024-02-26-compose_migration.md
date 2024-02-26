---
title: "[Compose Migration] 기존 앱에 compose 적용 시작하기"
excerpt: "컴포즈 적용 어디서부터 시작해야 할까??"
categories:
  - Compose
tag:
  - compose migration
  - compose
  - BaseComposeActivity
  - BaseComposeFragment
  - AndroidView
  - ComposeView
  - kotlinCompilerExtensionVersion

last_modifeid_at: 2024-02-26
toc: true
toc_sticky: true
search: true
---

## 🔗들어가며

새로 참여한 프로젝트에서 `compose migration`을 진행하게 되었다. xml -> compose로 바뀌고 있는 요즘 compose migration을 진행하고자 하는 분들에게 도움이 될 수 있도록 그 과정을 꼼꼼히 정리보고자 한다.

우선 migration을 진행하는 과정에서 큰 도움이 되었던 영상이 하나 있다. 2023 드로이드나이츠에서 직접 청강을 했었지만 그 당시에는 컴포즈를 활용 해본 경험이 없어 이해하면서 듣기 보다는 컴포즈가 이렇게 활용되구나를 느끼며 들었는데...

compose로 migration을 진행하게 되면서 [드로이드나이츠 해당 영상](https://www.youtube.com/watch?v=E4OmRPD332Q)을 정말 많이 참고했던 것 같다!!

<br>

## 🔗migration의 시작!!

우선 해당 게시글에서는 처음 compose를 도입할 때 진행하는 과정을 다루고자 한다.
- `의존성 추가`
- `새로운 activity / fragment compose로만 작성하기`
- `기존 레거시 뷰에 compose 적용하기`
- `컴포즈에서 AndroidView란??`

이렇게 4가지를 먼저 다뤄보고자 한다! 추후 기존 recyclerView를 LazyColumn으로 다루는 방법을 Compose 시리즈로 차근차근 정리할 것이다!

그리고 또한, 해당 게시글에서는 custom theme을 다루지 않는다. 이 내용도 별도의 글로 다룰 것이다! 그럼 추울발~~~~~

### 🔗 1. 의존성 추가
```kotlin
android {
    namespace = "???"

    buildFeatures {
        compose = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = libs.versions.kotlin.compiler.get()
    }
}
```

필자의 경우에는 `버전 카탈로그`를 활용해 모든 의존성 버전을 관리하고 있다. 따라서 위의 `kotlinCompilerExtensionVersion`의 버전도 버전 카탈로그를 통해 가져왔다.

그리고 `kotlinCompilerExtensionVersion`는 사용하는 `kotlin에 맞는 compose compiler 버전`을 선택해야 한다. 다음 [링크](https://developer.android.com/jetpack/androidx/releases/compose-kotlin?hl=ko#pre-release_kotlin_compatibility)를 참고하자!

이렇게 compose compiler만 별도의 버전으로 분리한 이유는 [다음 게시글](https://android-developers.googleblog.com/2022/06/independent-versioning-of-Jetpack-Compose-libraries.html)에서 확인할 수 있었다. kotlin 버전이 업그레이드 되면 그에 따라 compose compiler의 버전도 업그레이드 되어야 한다. 따라서 다른 compose 라이브러리와 별도로 버전을 관리하면서 조금 더 빠르게 버전 업그레이드를 하기 위해서이다. 

```kotlin
    val composePlatform = platform(libs.compose.bom)
    api(composePlatform)
    api(libs.compose.ui)
    api(libs.compose.ui.tooling)
    api(libs.compose.preview)
    api(libs.compose.activity)
    api(libs.compose.material)
    api(libs.compose.foundation)
    api(libs.compose.constraintlayout)
    api(libs.compose.appcompat.theme)
```

이제는 컴포즈 의존성을 추가해줄 것이다. 이때 우리는 <span style = "background-color:#fff5b1">['compose bom'](https://developer.android.com/jetpack/compose/bom?hl=ko)의 버전만 지정하여 나머지 compose 라이브러리 버전을 자동으로 지정할 수 있게 된다.</span> 물론 bom에서 제공하는 버전 이외의 버전을 수동으로 설정해 지정하는 것도 가능하다!

```kotlin
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-material = { module = "androidx.compose.material:material" }
compose-ui = { module = "androidx.compose.ui:ui" }
compose-ui-tooling = { module = "androidx.compose.ui:ui-tooling" }
compose-preview = { module = "androidx.compose.ui:ui-tooling-preview" }
compose-activity = { module = "androidx.activity:activity-compose" }
compose-foundation = { module = "androidx.compose.foundation:foundation" }
```

버전 카탈로그에서는 이렇게 정의되어 있다!!

<br>

## 🔗 2. 새로운 activity / fragment compose로만 작성하기!

우선 migration을 진행한다면 두 가지 경우일 것이다.
- <span style = "background-color:#fff5b1">기존 화면의 일부만 compose 적용!</span>
- <span style = "background-color:#fff5b1">새로 추가되는 화면 compose 활용!</span>

우선은 두번 째 경우를 먼저 살펴보고자 한다. 새로운 화면 전체를 <span style = "background-color:#fff5b1">compose로만</span> 구성하고자 한다면 `ComponentActivity`를 활용할 수 있다. 

아마 여러분은 기존에도 BaseAcitity나 BaseFragment를 만들어 작업했을 것이다. 이처럼 `BaseComposeActivity`를 만들어 작업하자!!

```kotlin
abstract class BaseComposeActivity: ComponentActivity() {
    ...

    @Composable
    abstract fun Content()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...
        setContent {
            Content()
        }
    }
}
```

정말 간단하다!! <span style = "background-color:#fff5b1">구성할 화면을 Content를 상속해 정의할 수 있도록 abstract로 @Composable 함수를 구성한다.</span>

그렇다면 우리는 다음과 같이 이를 활용할 수 있다.

```kotlin
@AndroidEntryPoint
class CreateSuggestionActivity: BaseComposeActivity() {
  ...

    @Composable
    override fun Content() {
        MaterialTheme {
            CreateSuggestionScreen()
        }
    }
}
```

정말 간단하지 않은가?? fragment도 비슷하다!

```kotlin
abstract class BaseComposeFragment: Fragment() {
    @Composable
    abstract fun Content()

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return ComposeView(requireContext()).apply { 
            setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed)
            setContent {
                Content()
            }
        }
    }
}
```

✨ 여기서 하나 확인해야 할 것이 <span style = "background-color:#fff5b1">`setViewCompositionStrategy`</span>이다. 이는 컴포즈를 언제쯤 release할 수 있는지에 대한 전략이다. 만약 화면은 보이지 않지만 컴포즈가 계속해서 렌더링을 시도하려고 한다면??? 메모리 릭등의 이슈들이 발생할 것이다. 따라서 적절한 시기에 해제해주는 전략을 지정해주는 것이다. 

전략은 다음과 같이 총 4가지가 존재하며 이를 다음 게시글에서 자세히 다루고 있다.
[언제 composition을 제거해야 할까?](https://915dbfl.github.io/compose/view_composition_strategy/)
- `DisposeOnDetachedFromWindow`
- `DisposeOnDetachedFromWindowOrReleasedFromPool`
- `DisposeOnLifecycleDestroyed`
- `DisposeOnViewTreeLifecycleDestroyed`

<br>

## 3. 기존 레거시 뷰에 compose 적용하기

처음 migration을 진행한다면 작은 요소부터 하나하나 compose화 해나가는 것이 좋다고 한다. 그렇다면 기존 xml 레거시 뷰에서는 compose를 어떻게 활용할 수 있을까??

정말 간단하다!! <span style = "background-color:#fff5b1">`ComposeView`</span>를 넣으면 된다!

우선 xml에 다음과 같이 선언해주고!

```kotlin
<androidx.compose.ui.platform.ComposeView
  android:id="@+id/compose_view"
  android:layout_width="match_parent"
  android:layout_height="match_parent"/>
```

이제 해당 xml의 activity나 fragment에서 composable 함수들로 화면을 구성해주면 된다!

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding.composeView.apply { 
            setContent {
                MaterialTheme {
                    Text(text = "")
                }
            }
        }
    }
```

binding을 통해 해당 composeView를 가져와 setContent만 적용하면 완성이다! 정말 간단하지 않은가?? 그렇다면 궁금할 것이다!! compose의 장점인 preview도 적용할 수 있을까??

간단하다! 아래와 같이 Composable 함수와 그에 대한 Preview를 정의한 후, xml 속 composeView의 `composableName`으로 지정해주면 된다.

```kotlin
@Composable
fun CustomText() {
    Text(text = "Hello, World!")
}

@Preview
@Composable
fun CustomTextPreview() {
    MaterialTheme {
        CustomText()
    }
}
```

```kotlin
        <androidx.compose.ui.platform.ComposeView
            android:id="@+id/compose_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            tools:composableName="com.android.MainActivity.CustomTextPreview"/>
```

</br>

## 🔗 4. 컴포즈에서 AndroidView란??

컴포즈로 화면을 구성하다보면 <span style = "background-color:#fff5b1">compose에서 지원하지 않는 기존 android widget을 활용하고 싶을 때</span>도 있다!! 이때 활용하는 것이 `AndroidView`이다!!

역시나 사용법이 정말 간단하다. <span style = "background-color:#fff5b1">넣고자 하는 위치에 AndroidView composable 함수를 활용해 factory에 원하는 widget을 넣어주면 된다.</span>

```kotlin
Column(...) {
      ...

        AndroidView(
            modifier = Modifier.fillMaxWidth(),
            factory = { context ->
                LGTMEditText(context).apply {
                    setLifecycleOwner(lifecycleOwner)
                    bindStateEditTextData(title)
                    setMaxLine(1)
                    onTextChangedListener {
                        updateTitleEditTextData()
                    }
                    onFocusChangedListener { isFocused ->
                        if (isFocused) {
                            coroutineScope.launch {
                                delay(420)
                                titleSectionBringIntoViewRequester.bringIntoView()
                            }
                        }
                    }
                }
            }
        )
    }
```

우선 여기까지 진행한다면 기존 프로젝트에서 어떻게 컴포즈를 활용할 수 있는지 대략적으로 감이 잡힐 것이다!! 이제 필요한 부분들을 하나씩 compose로 바꿔가며 적용해보자!

<br>

## 참고
* <https://developer.android.com/jetpack/androidx/releases/compose-kotlin?hl=ko#pre-release_kotlin_compatibility>
* <https://android-developers.googleblog.com/2022/06/independent-versioning-of-Jetpack-Compose-libraries.html>
* <https://developer.android.com/jetpack/compose/bom?hl=ko>
* <https://www.youtube.com/watch?v=E4OmRPD332Q>