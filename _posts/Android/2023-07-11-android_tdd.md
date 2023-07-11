---
title: "[TDD] 안드로이드에서 TDD"
excerpt: "Advanced Android in Kotlin 05.1: Testing Basics"
categories:
  - Android
tag:
  - android
  - test-driven-development
  - tdd

last_modified_at: 2023-07-11
toc: true
toc_sticky: true
search: true
---

<br>
이 글은 'Advanced Android in Kotlin 05.1: Testing Basics'를 진행하며 배운 내용을 기록한 글이다.

처음 해당 프로젝트를 실행할 때 <span style = "background-color:#fff5b1">'Flamingo' 환경에서 버전 이슈로 인해 빌드가 되지 않는 오류</span>가 있어 이를 해결해 해당 프로젝트에 pr을 날렸다. 만약 동일한 문제를 겪고 있다면 다음 방식을 한 번 시도해보길 추천한다!
* [✓Update version to resolve build issues for AndroidStudio, Flamingo](https://github.com/google-developer-training/advanced-android-testing/pull/293)

<br>

# 👩🏻‍💻 local vs instrumented test

우선 안드로이드 스튜디오에서 첫 프로젝트를 생성하면 자동으로 다음의 세 [소스세트](https://developer.android.com/studio/build?hl=ko#sourcesets)를 확인할 수 있다.

<img src = "https://drive.google.com/uc?id=10ux3_PXjBIMHdAv62J98MLUK95j2HN33" width = 500 height = 300>

여기서 우리가 가장 많이 다루는 것이 바로 가장 위에 있는 소스 세트일 것이다. 그것이 바로 앱의 코드가 들어있는 main 소스세트이다. 그리고 그 밑에 있는 `androidTest`의 경우, <span style = "background-color:#fff5b1">instrumented test(계측 테스트)</span>이다. 마지막으로 단순 `test` 폴더는 <span style = "background-color:#fff5b1">local test</span>에 해당한다.

그렇다면 instrumented test와 local test의 차이는 무엇일까? 이를 간단히 정리하면 다음과 같다.

* `instrumented test`
  * JVM에서 local하게 돌아간다.
  * 별도의 에뮬레이터나 실기기가 필요하지 않다.
  * 빠르지만 실제 실행과는 차이가 존재한다.
  * : <span style = "background-color:#fff5b1">Android JUnit</span> 사용
* `local test`
  * 에뮬레이터나 실기기에서 돌아간다.
  * 느리지만 실제 실행과 연관이 깊다.
  * : <span style = "background-color:#fff5b1">Android Instrumented Tests</span> 사용

<br>

# 👩🏻‍💻 코드로 확인하는 테스트

```kotlin
// A test class is just a normal class
class ExampleUnitTest {

   // Each test is annotated with @Test (this is a Junit annotation)
   @Test
   fun addition_isCorrect() {
       // Here you are checking that 4 is the same as 2+2
       assertEquals(4, 2 + 2)
   }
}
```

위 코드는 local test에 대한 코드이다. local test에서는 위에서 설명했듯이 <span style = "background-color:#fff5b1">`JUnit`</span>이 사용된다. 하나의 클래스가 안에 여러개의 테스트가 존재할 수 있다.(여기서 하나의 테스트만 실패해도 전체 테스트가 실패한 것으로 인식된다.) 각 테스트는 <span style = "background-color:#fff5b1">메소드</span>로 표현이 된다. test마다 <span style = "background-color:#fff5b1">`@Test`</span> 어노테이션이 붙게 되며, local test의 경우, 일반적으로 <span style = "background-color:#fff5b1">[assertion](https://simplecode.kr/16)을 포함한다. 여기서 어노테이션, assertion 모두 JUnit에 속해있다.</span>

```kotlin
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {
    @Test
    fun useAppContext() {
        // Context of the app under test.
        val appContext = InstrumentationRegistry.getInstrumentation().targetContext
        assertEquals("com.example.android.architecture.blueprints.reactive",
            appContext.packageName)
    }
}
```
그렇다면 이 코드를 봐보자! @Test 어노테이션이 없는 것을 통해 우선 위 메소드들과 형태가 다른 것을 확인할 수 있다. 이는 <span style = "background-color:#fff5b1">`Instrumented test`</span>이다. 그렇다면 local test와 Instrumented test를 구분짓는 요소는 무엇일까? 그것은 바로 <span style = "background-color:#fff5b1">`Android Os나 Android Framework code`</span>를 포함하느냐이다. 위 코드를 보면 <span style = "background-color:#fff5b1">Context, InstrumentationRegistry</span>을 사용하는 것을 알 수 있다. 따라서 위 테스트 코드는 Instrumented test에 해당하게 되는 것이다.

✓여기서 잠깐! 혹시 Insrumented test를 실행하다 다음의 오류를 발견한다면 아래 stackoverflow에 해결법이 나와 있으니 참고하자!
* <https://stackoverflow.com/questions/71513360/run-android-instrumented-tests-fail>

<br>

# 👩🏻‍💻 Test 작성 시 고려해야 할 부분
* 하나의 메소드에 대해 test를 작성하고자 할 때 해당 메소드에서 우클릭을 하면 쉽게 테스트를 추가할 수 있다.

<img src ="https://drive.google.com/uc?id=1UddCMLv2bsUkT1JuAiH4O3JC5nE2LvuZ" width = 500 height = 300>

테스트를 추가할 때 androidTest, Test 소스 세트 중 어디에 추가할 것인지를 고르게 한다. 만약 단순히 계산 결과를 확인하는 것이라면 local test에 해당하겠지만 Android왕 연관된 코드가 들어간다면 instrumented test에 추가해야 할 것이다.

* 테스트 이름은 뭘로 해야 할까?
  * (테스트 대상 메소드 이름)_(action이나 input)_(예상하는 결과)
  * ex) getActiveAndCompletedStats_noCompleted_returnsHundredZero

* 그렇다면 테스트 함수 안에는 어떠한 내용이 들어가야 할까?
  * <span style = "background-color:#fff5b1">Given, When, Then</span>
  * `Given`
    * 테스트를 위해 필요한 object나 state를 setup한다.
  * `When`
    * 위에서 만든 object를 테스트하기 위해 실행한다. 즉, 테스트 메소드를 call하는 것이다.
  * `Then`
    * assert function을 사용하여 결과를 check하는 부분이다.

그렇다면 다음 테스트 코드를 통해서 <span style = "background-color:#fff5b1">Given, When, Then</span> 구조를 명확히 이해하고 넘어가자.
```kotlin
    @Test
    fun getActiveAndCompletedStats_noCompleted_returnsHundredZer() {

        //Given: Create an active tasks
        val tasks = listOf<Task>(
            Task("title", "desc", isCompleted = false)
        )

        //When: Call your function
        val result = getActiveAndCompletedStats(tasks)

        //Then: Check the result
        assertThat(result.completedTasksPercent, `is`(0f))
        assertThat(result.activeTasksPercent, `is`(100f))
    }
```
<br>

결과적으로 이렇게 test를 쭉 구현하다 보면 다음과 같이 case 하나하나에 대한 test가 생기게 된다.

```kotlin
    @Test
    fun getActivityAndCompletedStats_both_returnsFortySixty() {
        val tasks = listOf(
            Task("title", "desc", isCompleted = true),
            Task("title", "desc", isCompleted = true),
            Task("title", "desc", isCompleted = true),
            Task("title", "desc", isCompleted = false),
            Task("title", "desc", isCompleted = false)
        )

        val result = getActiveAndCompletedStats(tasks)
        assertEquals(result.activeTasksPercent, 40f)
        assertEquals(result.completedTasksPercent, 60f)
    }

    @Test
    fun getActiveAndCompletedStats_error_returnsZeros() {
        // When there's an error loading stats
        val result = getActiveAndCompletedStats(null)

        // Both active and completed tasks are 0
        assertThat(result.activeTasksPercent, `is`(0f))
        assertThat(result.completedTasksPercent, `is`(0f))
    }

    @Test
    fun getActiveAndCompletedStats_empty_returnsZeros() {
        // When there are no tasks
        val result = getActiveAndCompletedStats(emptyList())

        // Both active and completed tasks are 0
        assertThat(result.activeTasksPercent, `is`(0f))
        assertThat(result.completedTasksPercent, `is`(0f))
    }
```

<br>

# 👩🏻‍💻 ViewModel에 대해 Test 진행하기

viewModel에 대해서도 Test를 진행할 수 있다! 그렇다면 Test를 만들기 직전에 해당 Test는 local, instrumented 중 어디에 들어가야 할까? ViewModel test의 경우, Android flatform code가 없기 때문에 <span style = "background-color:#fff5b1">test</span>에 들어가는 것이 일반적이다. 하지만 viewModel에 대해 Test를 진행하다보면 <span style = "background-color:#fff5b1">viewModel instance</span>을 얻을 때 <span style = "background-color:#fff5b1">ApplicationContext</span>이 필요하다는 것을 알 수 있다.

그럼 이때 어떻게 하면 좋을까? 이를 위해서 AndroidTest로 Test의 위치를 변경해야 할까? 이때 사용할 수 있는 라이브러리가 <span style = "background-color:#fff5b1">AndroidX Test</span>이다!

## ✓ AndroidX Test
* <span style = "background-color:#fff5b1">local test에서 simulated Android framework class들을 사용 가능하다.</span>
* local test뿐만 아니라 instrumented test에서도 사용이 가능하다.
  * 즉, 하나의 코드로 local/instrumented test 모두에서 활용이 가능하며, 환경에 따라 다르게 동작한다.
    * `local test`: <span style = "background-color:#fff5b1">simulated Android environment</span> 활용
    * `instrumented test`: <span style = "background-color:#fff5b1">실제 context</span>를 가져와 실행

<br>

## ✓ test 작성하기
1. dependency 추가
```kotlin
// AndroidX Test - JVM testing
testImplementation "androidx.test.ext:junit-ktx:$androidXTestExtKotlinRunnerVersion"
testImplementation "androidx.test:core-ktx:$androidXTestCoreVersion"
testImplementation "org.robolectric:robolectric:$robolectricVersion"
```
* 해당 라이브러리는 test 소스세트에서만 활용할 것이기 때문에 `testImplementation`을 통해서 dependency를 추가해준다.
* 여기서 version은 검색을 통해 최신 버전을 넣어주면 된다.
* ‼️ 여기서 `robolectric` 라이브러리 같은 경우, <span style = "background-color:#fff5b1">sdk 33을 지원해주는 버전인지를 확인</span>하고 해당 버전으로 설정해준다. 그게 아니라면 버전 오류가 발생한다.
  * `robolectric`은 <span style = "background-color:#fff5b1">local test에서 simulated android environment</span>을 제공해주는 library이다.


```kotlin
@RunWith(AndroidJUnit4::class)
class TasksViewModelTest {

    //AndroidX Test function에 해당
    @Test
    fun addNewTask_setNewTaskEvent() {
        val tasksViewModel = TasksViewModel(ApplicationProvider.getApplicationContext())
        tasksViewModel.addNewTask()
    }
}
```
* `RunWith`로 test runner를 지정해준다.
  * test runner는 JUnit 요소로 test runner가 없을 경우, test는 실행될 수 없다.
  * 지정을 안해줄 경우, default runner가 설정되고, `AndroidJUnit4`는 AndroidX Test를 local, instrumented 모두에서 제공하는 test runner이다.


즉, 정리를 하면 <span style = "background-color:#fff5b1">단순 local test에서 simulated android environment</span>을 사용하고자 한다면 `AndroidX Test function`을 정의해야 하고, <span style = "background-color:#fff5b1">사용되는 simulated android environment</span>은 `robolectric` 라이브러리에 의해 제공받을 수 있게 되는 것이다!

<br>

# 👩🏻‍💻 LiveData에 Test 적용하기
LiveData를 test하기 위해서는 크게 두가지 작업을 진행해야 한다. <span style = "background-color:#fff5b1">1. InstantTaskExecutorRule 사용, 2. LiveData observation</span>. 그렇다면 우선 InstantTaskExecutorRule이 하는 역할에 대해 알아보자!

## ✓InstantTaskExecutorRule
* JUnit Rule
* <span style = "background-color:#fff5b1">@get:Rule</span>을 통해서 InstantTaskExecutorRule 클래스의 코드들이 test 이전과 이후에 실행될 수 있도록 한다. 
*  동일한 쓰레드에서 architecture component와 관련된 백그라운드 작업이 진행돼 동기화에 신경을 쓰지 않아도 되게 된다. 따라서 <span style = "background-color:#fff5b1">livedata를 포함해 테스트를 진행하려고 한다면 rule을 사용하자!!</span>

```kotlin
testImplementation "androidx.arch.core:core-testing:$archTestingVersion"
```

우선 Rule을 사용하기 위해 `Architecture Components core testing library`를 추가한다.

그 다음 진행해야 할 부분은 바로 위에서 언급했듯이 <span style = "background-color:#ffff5b1">liveData observation</span>이다. 우리가 liveData를 observe하고 변경이 있을 때마다 관찰을 하기 위해서는 <span style = "background-color:#fff5b1">lifeCycleOwner</span>을 등록해주어야 한다. 하지만 <span style = "background-color:#fff5b1">local test를 진행한다면 owner로 등록할 activity나 fragment가 존재하지 않는다는 문제점이 있다.</span> 따라서 여기서 사용해야 할 것이 바로 계속해서 observe한다고 지정하는 <span style = "background-color:#fff5b1">observeForever 함수</span>이다. 

```kotlin
@Test
fun addNewTask_setsNewTaskEvent() {

    // Given a fresh ViewModel
    val tasksViewModel = TasksViewModel(ApplicationProvider.getApplicationContext())


    // Create observer - no need for it to do anything!
    val observer = Observer<Event<Unit>> {}
    try {

        // Observe the LiveData forever
        tasksViewModel.newTaskEvent.observeForever(observer)

        // When adding a new task
        tasksViewModel.addNewTask()

        // Then the new task event is triggered
        val value = tasksViewModel.newTaskEvent.value
        assertThat(value?.getContentIfNotHandled(), (not(nullValue())))

    } finally {
        // Whatever happens, don't forget to remove the observer!
        tasksViewModel.newTaskEvent.removeObserver(observer)
    }
}
```
* 여기서 주의할 점은 observeForever를 사용하면 말그대로 계속해서 관찰자가 등록되어 있는 것이므로 test가 끝난 후에는 <span style = "background-color:#fff5b1">관찰자 제거와 관찰자 누출의 위험</span>에 주의를 기울여야 한다.

위를 보면 하나의 liveData를 test할 때마다 observer가 생성되고, observeForever로 등록되고, 이를 제거하는 과정이 계속 반복될 것이다.(boilerplate) 따라서 이를 Extension으로 빼두는 것이 좋다!

```kotlin
// boilerplate 코드를 제거하기 위한 liveData extension
import androidx.annotation.VisibleForTesting
import androidx.lifecycle.LiveData
import androidx.lifecycle.Observer
import java.util.concurrent.CountDownLatch
import java.util.concurrent.TimeUnit
import java.util.concurrent.TimeoutException


@VisibleForTesting(otherwise = VisibleForTesting.NONE)
fun <T> LiveData<T>.getOrAwaitValue(
    time: Long = 2,
    timeUnit: TimeUnit = TimeUnit.SECONDS,
    afterObserve: () -> Unit = {}
): T {
    var data: T? = null
    val latch = CountDownLatch(1)
    val observer = object : Observer<T> {
        override fun onChanged(o: T?) {
            data = o
            latch.countDown()
            this@getOrAwaitValue.removeObserver(this)
        }
    }
    this.observeForever(observer)

    try {
        afterObserve.invoke()

        // Don't wait indefinitely if the LiveData is not set.
        if (!latch.await(time, timeUnit)) {
            throw TimeoutException("LiveData value was never set.")
        }

    } finally {
        this.removeObserver(observer)
    }

    @Suppress("UNCHECKED_CAST")
    return data as T
}
```

이를 활용한다면 위의 코드가 다음과 같이 간단해지는 것을 확인할 수 있다.

```kotlin
@RunWith(AndroidJUnit4::class)
class TasksViewModelTest {

    @get:Rule
    var instantExecutorRule = InstantTaskExecutorRule()


    @Test
    fun addNewTask_setsNewTaskEvent() {
        // Given a fresh ViewModel
        val tasksViewModel = TasksViewModel(ApplicationProvider.getApplicationContext())

        // When adding a new task
        tasksViewModel.addNewTask()

        // Then the new task event is triggered
        val value = tasksViewModel.newTaskEvent.getOrAwaitValue()

        assertThat(value.getContentIfNotHandled(), not(nullValue()))


    }

}
```

<br>

# 👩🏻‍💻 여러 test에서 사용되는 값의 경우?

하나의 클래스 안에 여러개의 test가 존재하고, 그러한 test들은 모두 독립적으로 실행된다. 그렇다면 여러 test에서 사용되는 값의 경우, 어떻게 선언하고 사용할 수 있을까? 아래 코드를 한 번 확인해보자!

```kotlin

@RunWith(AndroidJUnit4::class)
class TasksViewModelTest {

    @Test
    fun addNewTask_setNewTaskEvent() {
        val tasksViewModel = TasksViewModel(ApplicationProvider.getApplicationContext())

        ...

    }

    @Test
    fun setFilterAllTasks_tasksAddViewVisible() {
        //Given a fresh ViewModel
        val viewModel = TasksViewModel(ApplicationProvider.getApplicationContext())

      ...

    }
}
```

코드를 보면 <span style = "background-color:#fff5b1">두 test에서 모두 viewModel이 사용된다.</span> 그럼 하나의 viewModel을 선언하고 이를 여러 test에서 사용하면 참 좋을 것이다. 이때 사용하는 것이 바로 `@Before 어노테이션`이다. 말 그대로 test 전에 할 일을 @Before 어노테이션을 붙여 선언해주면 된다.(@After 어노테이션도 존재한다.) 코드가 비교적 이해하기 쉬어 코드를 보며 확인해보자!

```kotlin
@RunWith(AndroidJUnit4::class)
class TasksViewModelTest {
    private lateinit var tasksViewModel: TasksViewModel

    @get:Rule
    var instantTaskExecutorRule = InstantTaskExecutorRule()


    @Before
    fun setupViewModel() {
        tasksViewModel = TasksViewModel(ApplicationProvider.getApplicationContext())
    }

    @Test
    fun addNewTask_setNewTaskEvent() {
      ...

    }

    @Test
    fun setFilterAllTasks_tasksAddViewVisible() {
      ...
    }
  }
```
클래스 안에 공통으로 쓰일 viewModel 변수를 lateinit으로 선언해준다. 이를 @Before 어노테이션이 붙은 setup 함수에서 초기화를 해주고, 여러 테스트에서 편리하게 사용하면 된다!

<br>

# 😌 정리

항상 TDD를 적용해야 한다는 말만 듣고, 공부를 해볼 엄두가 나지 않았다. 이번에 안드로이드 코드랩을 통해 TDD를 학습하면서 안드로이드에서 어떻게 TDD를 적용해야 하는지 명확히 이해할 수 있게 된 것 같다. 소마에서 프로젝트를 본격적으로 진행하기 앞서 TDD를 한 번 쭉 훑어보길 잘 한 것 같다! 이번 기회에 한 번 TDD를 적용해봐야겠다!! 아직 완벽하게 이해한 것 같지는 않지만 직접 적용해보면서 더 깊게 파헤쳐봐야겠다! TDD가 처음이라면 아래 코드랩을 통해 쭉 진행해보는 것을 추천한다! 뭔가 큰 틀이 잡아지는 듯하다!

<br>

## 📃참고
* <https://developer.android.com/codelabs/advanced-android-kotlin-training-testing-basics#0>

