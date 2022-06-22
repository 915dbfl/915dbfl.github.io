---
title: "[Kotlin] Coroutine"
excerpt: "코루틴을 통핸 비동기화 처리"
categories:
  - Kotlin
tag:
  - kotlin
  - suspend

last_modified_at: 2022-06-22
toc: true
toc_sticky: true
search: true
---

<br>

🙄 코루틴에 대해 본격적으로 알아보기 전에 먼저 동기화/비동기화가 무엇인지 알아보자!

<br>

# 🙋‍♀️ 동기화 vs 비동기화

* 동기화(Synchronous): **하나의 작업이 완료될 때까지** 다른 작업을 수행하지 않고 **기다리는 방식**
  * "요청한 결과가 요청한 시점에 주어져야 한다."
* 비동기화(Asynchronous): 하나의 작업이 끝날 때까지 기다리지 않고 다른 작업도 진행하는 방식
  * "요청의 결과가 요청한 시점 동시에 주어지지 않는다."

<br>

## ➕ 비동기화가 필요한 이유?
* 애플리케이션이 시작과 동시에 실행이 되는 것이 **Main Thread**이다. 여기서는 안드로이드 프래임워크 속 앱의 동작에 필요한 준비를 진행한다.
  * [main thread와 thread에 대해 더 알고 싶다면?☝️](https://recipes4dev.tistory.com/143?category=768056)

* 만약 ui를 그리는 과정에서 10초 이상이 걸리는 작업이 존재한다고 생각하자. 이 경우, 비동기처리를 하지 않으면 main thread는 10초가 걸리는 해당 작업으로 인해 원활한 작업이 어려워진다.

따라서 main Thread에서는 실행 시간이 긴 작업, 오랜 대기와 같은 무거운 작업은 피해야 하며, 이러한 것을 처리하기 위해서 **비동기 처리**가 필요하게 된다.

<br>

# 🙋‍♀️ Coroutine

> [Android의 Kotlin 코루틴](https://developer.android.com/kotlin/coroutines?hl=ko#handling-exceptions)

비동기 처리 방식에는 무척 다양한 방법이 있다.
현재는 deprecated된 AsyncTask나 다수의 쓰레드를 관리하는 방법이 있으나 Coroutine의 경우, 훨 효율적으로 비동기 처리를 진행할 수 있다.

코루틴은 쓰레드 위에서 실행이 된다. 만약 task1이 실행되던 중 task2가 실행되더라도 **기존 스레드를 유지하며** task1이 저장된 후, task2가 실행되게 된다. task1이 다시 실행될 때는 **저장된 위치에서 다시 실행된다.** 이러한 점으로 coroutine을 사용할 경우, 다수의 스레드를 사용하는 것보다 훨 적은 자원이 소모되게 된다.

<br>

# 🙋‍♀️ Coroutine의 형태

coroutine을 사용하기 위해 알아야 하는 용어는 크게 **coroutineScope, coroutineContext, coroutineBuilder**로 세 가지가 존재한다.

```kotlin
CoroutineScope(CoroutineContext).CoroutineBuilder{
  ...
}
```

여기서 저 세 가지 구성요소를 통해서 코루틴이 진행되는 흐름을 한 문장으로 표현한다면 다음과 같다!

> context를 통해 scope가 생성되고, builder로 scope 안에서 실행!

<br>

## ☝️ coroutineScope

coroutinesScope는 코루틴을 제어할 수 있는 **범위(scope)**을 지정하는 역할을 한다.

어떠한 scope를 사용하느냐에 따라서 작업이 취소되고, 끝나는 지점이 달라지게 된다.

1. CoroutineScope: 사용자 지정
* 가장 기본이 되는 방식이다.
* 필요 시마다 선언을 해주며, **필요가 없어지면 종료**가 된다.
> 코루틴가 정의된 액티비가 종료되는 경우, 당연히 코루틴도 필요가 없어지므로 **종료**가 된다.

2. GlobalScope: 앱 실행 - 앱 종료
* 앱 실행 - 앱 종료 시점까지 실행시킬 수 있다.
> 액티비티가 종료된다 하더라도 코루틴이 **완료**될 때까지 동작한다.

👌 보통 필요 시마다 수행되는 코루틴의 경우, **CoroutineScope**을 사용하는 것이 권장된다!


## ✌️coroutineContext

코루틴이 실행될 **맥락(context)**를 지정해주는 역할을 한다.

즉, 코루틴을 통해 실행할 코드에 따라 다양한 맥락을 지정하게 된다.
coroutineContext로는 크게 3가지 종류가 존재한다.

1. **Dispatcher.Main**: Ui를 구성하는 작업용으로 예약된 스레드에서 실행한다.
  * **main thread**에서 코루틴을 진행하며, **ui와 상호작용하기**에 최적화!
2. **Dispatcher.IO**: 코루틴을 i/O 작업용으로 예약된 스레드에서 실행한다.
  * 읽고 쓰는 작업을 진행!
3. **Dispatcher.Default**: 기본 스레드 풀
  * 복잡한 연산 등 **CPU를 많이 쓰는 작업**에 최적화!

이처럼 Dispatcher는 코루틴을 적당한 스레드에 할당하게 된다.


## 👌 coroutineBuilder

cooutineBuilder를 통해서 코루틴을 실행시켜줄 함수를 지정하게 되고, 이는 크게 두가지 종류가 존재한다.

두가지 종류 모두 동일한 기능을 하게 되지만 반환하는 객체가 다르다!

1. launch{}
* **Job 객체**를 반환한다.
* 반환된 Job 객체의 경우, .join()함수를 통해서 해당 코루틴이 완료될 때까지 기다릴 수 있다.

2. async{}
* **Deferred 객체**를 반환한다.
* 

🙄 그렇다면 여기서 간단하게 job과 deferred 객체의 차이점에 대해서 간단히 짚고 넘어가자!

Job은 중간에 취소하거나 코루틴 블록이 완료될 때까지 기다리는 동작이 가능한 객체이다. Deferred 또한 job의 일종으로, 한마디로 결과값을 가지고 있는 job이다.

<br>

코루틴의 구성을 하나씩 살펴봤으니 이제 그 사용법에 대해서 자세히 살펴보자!!

<br>

# ➕ coroutine의 부모-자식 관계

코루틴의 경우, 부모-자식 관계가 존재한다.

> **부모 코루틴이 취소되면 자식 코루틴도 취소되며, 부모 코루틴은 자식 코루틴이 완료될 때까지 대기하게 된다.**

그렇다면 부모-자식 관계는 어떻게 파악할까? 단순히 **coroutineScope{}안에 정의된 코루틴**이라고 해서 부모-자식 관계라고 정의할 수 있을까??

🙄그에 대한 대답은 "No"이다.

```kotlin
        CoroutineScope(Dispatchers.Main).launch {//부모
            val job = Job()
            CoroutineScope(Dispatchers.IO + job).launch {//코루틴1
                Log.d("Coroutine1", "start")
                delay(2000)
                Log.d("Coroutine1", "end")
            }
            CoroutineScope(Dispatchers.IO).launch {//코루틴2
                Log.d("Coroutine2", "start")
                delay(1500)
                Log.d("Coroutine2", "end")
            }
            delay(200)
            Log.d("parent", "cancel")
            job.cancel()
            Log.d("finish", "finish done")
        }
```

위 코드를 확인해보면 하나의 코루틴 안에 다수의 코루틴이 존재하는 것을 확인할 수 있다.

여기서 결과를 한 번 예측해보자!!
만약 코루틴 1, 2가 모두 **parent의 자식이라면** <u>두 코루틴의 end는 찍히지 말아야 한다.</u>

하지만 결과는 다음과 같다!

```
D/Coroutine1: start
D/Coroutine2: start
D/parent: cancel
D/finish: finish done
D/Coroutine2: end
```

결과를 통해서 확인할 수 있듯이 코루틴2는 parent의 자식이 아닌 것을 확인할 수 있다. 그 이유는 **CroutineScope이 서로 다르기 때문이다.**

```kotlin
            CoroutineScope(Dispatchers.IO).launch {//코루틴2
                Log.d("Coroutine2", "start")
                delay(1500)
                Log.d("Coroutine2", "end")
            }
```

🙄 **두 번째 코루틴**은 부모의 **scope와 상관없이 새로운 CoroutineScope를 생성해 실행이 된다.** 따라서 부모가 cancel이 되어도 범위가 다르기 때문에 영향을 받지 않는다!

```kotlin
            CoroutineScope(Dispatchers.IO + job).launch {//코루틴1
                Log.d("Coroutine1", "start")
                delay(2000)
                Log.d("Coroutine1", "end")
            }
```

🙄 **첫 번째 코루틴**의 경우, 범위에 정의되어 있는 **+job**으로 인해서 **부모의 영향을 받게 된다.** 따라서 부모가 cancel이 되면서 코루틴 1도 취소되게 된다.

<br>

# ➕ coroutine 사용법

안드로이드에서는 코루틴을 생성할 때는 Acitivity에 CoroutineScope를 상속해 이를 life cycle로 하는 것을 권장하고 있다.

다음의 코드를 보면 무슨 말인지 정확히 이해할 수 있을 것이다!

```kotlin
class MainActivity(override val coroutineContext: CoroutineContext) : AppCompatActivity(), CoroutineScope {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        job = Job()
 
        launch {
            ..//코루틴 실행
        }
    }

    override fun onDestroy() {// 액티비티가 종료될 때 코루틴도 취소되게 설정한다.
        super.onDestroy()
        job.cancel()
    }
}
```

1. Activity에서 **CoroutineScope interface**를 상속받는다.
  * 이때 **coroutineCotex**t는 필수적인 요소이다.
2. Acitivity안에서 launch{}, async{}등을 **바로** 호출하며 코루틴을 실행한다.
3. 액티비티가 종료될 때 호출되는 **onDestroy()**에 **`job.cancel()`**를 선언하여 **액티비티가 종료될 때 실행되던 코루틴도 종료되게 한다.**

<br>


🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

## 📃참고
* <https://developer.android.com/kotlin/coroutines?hl=ko#handling-exceptions>
* <https://velog.io/@slobber/%EB%8F%99%EA%B8%B0%EC%99%80-%EB%B9%84%EB%8F%99%EA%B8%B0%EC%9D%98-%EC%B0%A8%EC%9D%B4>
* <https://goni95.tistory.com/117>
* <https://whyprogrammer.tistory.com/596>
* <https://oasisfores.com/kotlin-android-coroutine/>
  * 코루틴을 이해하는데 정말 도움이 많이 되었습니다!
* <https://stanleykou.tistory.com/entry/Kotlin-coroutine-Job-or-Deferred>