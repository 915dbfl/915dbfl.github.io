---
title: "[Kotlin Coroutine] Coroutine Exceptions Handling"
excerpt: "coroutine에서 exception이 발생하면 어떻게 처리되고, 또 처리해야 할까?"
categories:
  - Android
tag:
  - kotlin coroutine
  - kotlin coroutine exception
  - coroutine exception handling

last_modified_at: 2025-01-24
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가기 전
현재 우리 프로젝트에서는 coroutine을 활용하고 있지만, 코루틴 내부에서 예외가 발생했을 때 미처 처리되지 않는 예외에 대해서는 별도의 처리를 진행하고 있지 않다. 

8주간 급하게 구현을 마무리하고 현재는 프로젝트 리팩토링을 진행하고 있는데, 최근에는 구현 단계에서 놓친 부분들을 하나씩 학습하고 개선해 나가고 있다.

viewModel에 존재하는 string resource id를 어떻게 하면 효과적으로 관리할 수 있을지 고민하는 것부터 시작해, 이를 해결하기 위해서는 예외 처리가 개선되어야 한다고 느끼게 되었다. 그 과정에서 프로젝트 당시 가장 신경쓰지 못했던 `coroutine Exception`에 대해 공부를 하게 되었다. 

이 게시글은 공식 문서 [`Coroutine Exception Handling`](https://kotlinlang.org/docs/exception-handling.html#coroutineexceptionhandler)를 읽고 핵심적인 부분을 정리한 글이다. 서로 관련이 깊은 내용은 하나의 카테고리로 묶어 정리를 했기 때문에 공식 문서와는 내용 순서가 서로 다를 수 있다. 

그럼 시작해보자!

<br>

# 🔗 Exception propagation

우선 가장 처음으로 다룰 주제는 예외 전파이다. 코루틴은 계층적인 구조를 가진다. 그렇다면 그 속에서 예외가 발생한다면 그 예외는 부모까지 전파될까? 

이는 상황에 따라 다르다. 여기서는 코루틴 빌더인 `async`, `launch`에 따라 만들어진 코루틴에 대해 다룰 것이다. 그리고 각각에 의해 생성된 코루틴이 `root인지 아닌지`에 따라 예외 전파 여부를 확인해 보고자 한다.

# 🔗 일반적인 경우: not root

우선 일반적인 경우에 `launch`, `async`를 통해 생성된 coroutine의 예외는 어떻게 처리될까?

코드를 통해 하나씩 확인해보자.

## 1. launch

```kotlin
fun main() = runBlocking { // main 함수 추가
    launch {
        println("Throwing exception from launch")
        throw Exception("custom exception")
    }
    yield()
    println("not reached")
}
```

이를 실행시켜 본다면 자식 코루틴에서 던져진 예외에 의해 프로그램이 종료되는 것을 알 수 있다. 이처럼 <span style = "background-color:#fff5b1">launch는 기본적으로 부모에게 예외를 전파시킨다.</span>

(여기서 yield()는 실행을 양보하는 역할을 한다. 즉, 명시적으로 자식 코루틴의 출력 코드가 먼저 실행될 수 있도록 하는 것이다.)

```bash
Throwing exception from launch
Exception in thread "main" java.lang.Exception: custom exception
at FileKt$main$1$1.invokeSuspend (File.kt:6) 
at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith (ContinuationImpl.kt:33) 
at kotlinx.coroutines.DispatchedTask.run (DispatchedTask.kt:108)
```

## 2. async

```kotlin
fun main() = runBlocking<Unit> {  
    val deferred: Deferred<String> = async(CoroutineName("Coroutine1")) {  
        throw ArithmeticException()
    }

    try {
        deferred.await()
        println("not reached")
    } catch(e: ArithmeticException) {
        println("catch $e")
    }
}  
```
이를 실행시켜 본다면 async로 생성한 자식 코루틴에서 던진 예외에 의해서 main 함수가 종료되는 것을 확인할 수 있다. 

async는 기본적으로 <span style = "background-color:#fff5b1">await()를 호출할 때 예외가 노출된다.</span> await()을 불렀을 때 deferred job이 성공적으로 완료되었다면 결과값을 가지고 있는 Deferred 객체가, 그게 아니라면 예외가 던져진다.

하지만 <span style = "background-color:#fff5b1">await()를 부를 때 값이 "노출"이 되는 것이지, 예외는 launch와 마찬가지로 전파된다.</span>

```bash
catch java.lang.ArithmeticException
Exception in thread "main" java.lang.ArithmeticException
 at FileKt$main$1$deferred$1.invokeSuspend (File.kt:5) 
 at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith (ContinuationImpl.kt:33) 
 at kotlinx.coroutines.DispatchedTask.run (DispatchedTask.kt:108) 
```

## 3. 요약
일반적인 경우에 launch와 async에서 발생한 예외는 <span style = "background-color:#fff5b1">부모에게 전파가 된다.</span> 

자식에서 발생한 예외가 `언제 노출 되느냐`는 차이가 존재한다. launch는 예외가 던져지는 시점에, async는 await가 불릴 때 각각 예외가 노출 및 전파된다.

<br>

## 🔗 root coroutine인 경우

그렇다면 `launch`, `async`에 의해 생성한 코루틴이 root coroutine일 경우에는 각각 어떻게 처리가 될까?

우선 각 경우 어떻게 처리되는지 개념적으로 이해한 후에 코드를 살펴보자!

## 1. launch

<span style = "background-color:#fff5b1">launch에 의해 생성된 root coutine에서 예외 발생 시 Thread.uncaughtExceptionHandler에 의해 처리되는 uncaught 예외로 취급이 된다.</span>

이때 uncaught 예외의 경우, 말그대로 처리되지 않은 예외를 의미하며, 이런 예외는 최종적으로 Thread.uncaughtExceptionHandler의 구현체에서 처리가 되어진다.

따라서 해당 핸들러에 의해 예외가 잡혀 콘솔창에 예외 관련 정보가 출력될 뿐 프로그램이 종료되지 않는다.

## 2. async

async는 위에서도 말했듯이 예외가 바로 노출되지 않는다. `await()`를 호출할 때 비로소 예외가 노출된다.

그리고 만약 async에서 생성된 coroutine이 root라면 일반적인 경우와 동일하지만 <span style = "background-color:#fff5b1">부모에 예외를 전파하지는 않는다.</span>


아래 코드의 실행 결과를 통해 각 상황을 자세히 이해하고 넘어가자.

```kotlin
@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    val job = GlobalScope.launch { // root coroutine with launch
        println("Throwing exception from launch")
        throw IndexOutOfBoundsException() // Will be printed to the console by Thread.defaultUncaughtExceptionHandler
    }
    job.join()
    println("Joined failed job")
    val deferred = GlobalScope.async { // root coroutine with async
        println("Throwing exception from async")
        throw ArithmeticException() // Nothing is printed, relying on user to call await
    }
    try {
        deferred.await()
        println("Unreached")
    } catch (e: ArithmeticException) {
        println("Caught ArithmeticException")
    }
    
		print("finish main")
}
```

위 코드의 한줄한줄을 따라가며 최종적인 실행 결과를 예상해보자.
1. `GlobalScope`를 사용해 launch의 root coroutine 생성
2. 1번에서 생성한 코루틴 job에 대해 join
    - 이로 인해 1번 코루틴이 실행이 끝날 때까지 기다리게 된다. 이때 launch 안에서는 출력문 하나가 출력되고 예외가 던져진다.
    - 여기서 던진 예외는 위에서 말했듯이 `uncaught exception` 으로 취급되기 때문에 `Thread.uncaughtExceptionHandler` 에 의해 예외 처리가 돼 콘솔창에 예외에 대한 정보가 출력되고 다음 줄이 진행된다.
    
3. `GlobalScope`를 사용해 async의 root coroutine 생성
4. 3번에서 생성한 코루틴에 대해 await    
    - 이로 인해 3번 코루틴이 실행이 끝날 때까지 기다리게 된다. 하지만 await에 의해 내부적으로 잡힌 ArithmeticException 예외가 던져지며, try-catch로 해당 예외가 처리된다. 이때 예외 전파는 일어나지 않으며 마지막 print문이 출력된다.
5. main이 종료된다.

```bash
Throwing exception from launch
Exception in thread "DefaultDispatcher-worker-1 @coroutine#2" java.lang.Exception: custom exception
	at FileKt$main$1$job$1.invokeSuspend(File.kt:7)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:108)
	at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:584)
	at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:793)
	at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:697)
	at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:684)
	Suppressed: kotlinx.coroutines.internal.DiagnosticCoroutineConbashException: [CoroutineId(2), "coroutine#2":StandaloneCoroutine{Cancelling}@6ffdb4b3, Dispatchers.Default]
Joined failed job
Throwing exception from async
Caught ArithmeticException
finish main
```

## 3. 결론

root coroutine일 경우, launch / async 모두 에외 전파가 일어나지 않는다. launch의 경우, 발생한 exception이 `uncaught exception`으로 취급되어 hanlder에 의해 처리가 되며, async의 경우에는 명시적으로 예외 처리를 진행해 주어야 한다.

<br>

# 🔗 coroutineExceptionHandler 

## 1. coroutineExceptionHandler는 언제 사용될까?

uncaught exception을 처리하는 방식을 커스터마이징할 수 있다. 이때 활용되는 것이 `coroutineExceptionHandler`이다. (`Thread.uncaughtExceptionHandler`와 유사하다, 핵심은 <span style = "background-color:#fff5b1">uncaught exception을 처리하기 위해 사용된다는 것이다!</span>)

그렇다면 coroutineExceptionHandler를 꼭 사용해야 할까? 그건 아니다. 다음과 같은 상황에는 coroutineExceptionHandler이 의미가 없다.

- 모든 자식 코루틴의 예외가 부모에서 처리될 경우
- async를 사용할 경우(내부적으로 발생한 예외를 모두 잡아 Deferred 객체로 나타냄)

그렇다면 다음의 코드 실행 결과를 예상해보자.

```kotlin
val handler = CoroutineExceptionHandler { _, exception -> 
    println("CoroutineExceptionHandler got $exception") 
}
val job = GlobalScope.launch(handler) { // root coroutine, running in GlobalScope
    throw AssertionError()
}
val deferred = GlobalScope.async(handler) { // also root, but async instead of launch
    throw ArithmeticException() // Nothing will be printed, relying on user to call deferred.await()
}
joinAll(job, deferred)
```

우선 luanch, async 모두에서 예외가 발생되어 진다. 여기서 두 경우 모두 GlobalScope에 의해 root coroutine으로 실행되고 있다. 이때 CoroutineExceptionHandler는 GlobalScope에 등록되어 있기 때문에 동일한 Scope에서 생성된 job, deferred에서 처리되지 않은 예외가 발생한다면 해당 handler에서 잡아 처리하게 될 것이다.

우선 lauch에서 발생한 예외는 `uncaught exception`으로 취급되며, 별다른 예외처리가 진행되지 않고 있다. 따라서 등록한 coroutineExceptionHandler에 의해 처리가 돼 exception이 콘솔에 출력될 것이다.  하지만 async에 의해 실행된 코루틴의 경우는 async 내부적으로 예외가 잡혀 Deferrred 객체로 표현되기 때문에 coroutineExceptionHandler에 아무 영향을 미치지 않는다.

```bash
CoroutineExceptionHandler got java.lang.AssertionError
```

## 2. coroutineExceptinoHandler 예외 처리 시점 톺아보기

```kotlin
val handler = CoroutineExceptionHandler { _, exception -> 
    println("CoroutineExceptionHandler got $exception") 
}
val job = GlobalScope.launch(handler) {
    launch { // the first child
        try {
            delay(Long.MAX_VALUE)
        } finally {
            withConbash(NonCancellable) {
                println("Children are cancelled, but exception is not handled until all children terminate")
                delay(100)
                println("The first child finished its non cancellable block")
            }
        }
    }
    launch { // the second child
        delay(10)
        println("Second child throws an exception")
        throw ArithmeticException()
    }
}
job.join()
```

다음의 상황에서 coroutineExceptionHandler가 언제 작동하는지 예상해보자. 

1. first / second child 실행
2. first는 delay로 코루틴이 일시 중지 + second는 출력 후 exception 던짐

    - 그렇다면 그 다음 처리는 어떻게 될까? handler에 의해서 예외가 처리될까? 아니다. 결과는 다음과 같다.

1. exception에 의해 first child의 finally 실행
2. job.join에 의해 withConbash(NonCancellable)가 끝날 때까지 대기
    
    - NonCancellable에 의해 cancellation과 상관없이 실행되는 job을 통해 block내 코드들이 실행됨
    
3. join()이 완료된 후, coroutineExceptionHandler에 의해 비로소 ArithmeticException 예외가 처리됨

즉, <span style = "background-color:#fff5b1">부모에서의 예외 처리는 자식이 모두 종료된 후에 진행된다는 것을 명심하자.</span>

## 3. 자식 a/b/c 모두 예외 발생! 누굴 처리하지? (Exception aggregation)

위에서 coroutineExceptionHandler는 부모가 모두 종료된 후에 처리되지 않은 예외가 남았을 때 작동한다는 것을 알 수 있었다. 그렇다면 여러 자식한테서 예외가 발생한다면 coroutineExceptionHandler에서는 어떤 예외를 처리할까?

결론은 `첫 번째 자식의 예외`를 처리한다.

```kotlin
import kotlinx.coroutines.*
import java.io.*

@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception with suppressed ${exception.suppressed.contentToString()}")
    }
    val job = GlobalScope.launch(handler) {
        launch { // first child
            try {
                delay(Long.MAX_VALUE) // it gets cancelled when another sibling fails with IOException
            } finally {
                throw ArithmeticException() // the second exception
            }
        }
        launch { // second child
            delay(100)
            throw IOException() // the first exception
        }
        delay(Long.MAX_VALUE)
    }
    job.join()  
}
```

위의 코드를 살펴본다면  second child의 IOException이 먼저 발생하고 이로 인해 first child의 ArithmeticException이 발생하게 된다. 그리고 각 자식은 예외에 의해 취소가 되며 이 예외는 등록된 CoroutineExceptionHandler에게 가게 된다. 출력한 결과를 확인해본다면 handler에 의해 IOException만이 전달된 것을 확인할 수 있다.

```bash
CoroutineExceptionHandler got java.io.IOException with suppressed [java.lang.ArithmeticException]
```

<br>

# 🔗 CancellationException

<span style = "background-color:#fff5b1">CancellationException 예외는 모든 핸들러에서 무시가 된다.</span> 그렇다면 왜 그럴까? CancellationException은 발생하면 다른 예외들과 마찬가지로 코루틴이 종료된다. 하지만 다른 예외들과 달리 <span style = "background-color:#fff5b1">정상적인 흐름에서도 발생할 수 있다.</span> 따라서 다른 예외 상황과는 다르게 처리되어야 하며, 만약 catch를 통해 해당 예외를 잡더라도 예외 처리가 목적이 아닌 CancellationException의 정보를 디버깅 하는 용도여야 한다. 

여기서 CancellationException이 단순히 코루틴 취소 상황에서 발생하는 예외라고 생각하면 안된다. 공식 문서에서 나와 있듯이 `cancellable suspending functions` 가 발생시키는 것이다. cancellable suspending functions는 내부적으로 suspendCancellableCoroutine에 의해 구현된 것들로, suspend 상태에서 job이 cancel / complete 되었거나 명시적으로 cancel이 호출될 때 CancellationException을 던진다. 대표적인 예로는 `delay`, `withConbash`, `withTimeout` 등이 존재한다.

`job.cancel`에 의해 코루틴이 cancel이 되면, <span style = "background-color:#fff5b1">해당 코루틴은 취소되지만, 부모는 취소되지 않는다.</span> 아래의 코드를 한 번 살펴보자.

```kotlin
val job = launch {
    val child = launch {
        try {
            delay(Long.MAX_VALUE)
        } finally {
            println("Child is cancelled")
        }
    }
    yield()
    println("Cancelling child")
    child.cancel()
    child.join()
    yield()
    println("Parent is not cancelled")
}
job.join()
```
처음 해당 코드를 봤을 때 `yield`가 존재하는 이유를 잘 이해하지 못했다. 하지만 각각의 yield를 지워 실행시켜 본 결과 왜 해당 위치에 yield가 존재해야 하는지 이해할 수 있었다. 이 부분을 포함해서 위의 코드를 한 번 해석해보겠다.

1. 자식 코루틴 생성
2. yield로 자식 코루틴에게 실행 양보
    
    - 여기서 yield로 자식 코루틴에게 실행을 양보하는 이유는 해당 yield가 없을 경우, 자식이 아예 실행되지 않은 상태에서 자식의 cancel이 호출될 수 있다. 
    
    - 위에서 말했듯이 delay가 suspending 중에 cancel이 발생하게 되면  CancellationException이 발생하게 되는데, 만약 자식이 실행조차 되지 않는다면 해당 exception 역시 발생하지 않게 된다.
    
    - 따라서 finally는 실행조차 되지 않는다. 따라서 명시적으로 자식이 먼저 실행된 후에 부모가 실행될 수 있도록 우선 자신에게 잠깐 CPU를 양보하는 것이다.
    
3. child를 취소하고 기다리기
    
    - cancel, join으로 자식이 취소된 후 완료될 때까지 부모는 기다리게 된다. 여기서 delay를 실행 중인 자식이 취소가 되면서 CancellationException이 발생하게 되고, child의 finally 문이 실행되게 된다.
    
4. yield를 통해 잠시 실행 양보했다가 출력문 출력!
    
    - 해당 yield를 지워 실행시켜본다면 실행 결과가 차이가 없다. 즉, 큰 의미가 있는 코드는 아니다. 단순히 명시적으로 부모 코루틴에서 출력문을 출력하기 전에 잠깐 쉬며 자신이 출력할 것을 알리는 것이라고 생각하자.

```bash
Cancelling child
Child is cancelled
Parent is not cancelled 
```

여기서 기억할 것은 <span style = "background-color:#fff5b1">자식을 취소해도 부모는 취소가 되지 않는다는 것이다. 즉, 자식에서 취소가 된다면 CancellationException이 발생하게 되는데 이는 해당 코루틴만을 취소시키고 부모 코루틴에는 전파가 되지 않는다.</span>

```kotlin
val job = launch {
    val child = launch {
       	throw CancellationException("throw cancellationException from child")
    }
    yield()
    child.join()
    yield()
    println("Parent is not cancelled")
}
```
```bash
// console 결과
Parent is not cancelled
```
다음과 같이 자식에서 `CancellationException`이 발생하더라도 부모는 영향을 받지 않는다.

<br>

# 🔗 Supervision

위에서 다뤘듯이, <span style = "background-color:#fff5b1">코루틴 cancellation는 전체 계층을 통해 전파되는 양방향 관계이다.</span> 즉, 자식이 예외가 발생해 취소가 된다면 예외가 부모와 다른 코루틴에게도 전파되어 취소가 일어나게 된다.

<span style = "background-color:#fff5b1">하지만 단방향 cancellation만이 필요한 경우가 있다.</span> UI Component에 대한 job을 생각해보자. 해당 job이 실패했을 때와 취소/소멸될 때를 생각해보자.

1. ui component job의 실패 상황
    
    - 하나의 component에서 실패가 발생해도 다른 컴포넌트는 정상적으로 그려지는 것이 더 이상적인 동작이다. (단방향으로 영향을 미쳐야 한다.)
    
2. ui component job의 취소/소멸 상황
    
    - 하나의 ui component의 소멸이 일어날 경우, 화면 자체가 무의미하다. 이 때는 모든 ui component가 소멸되는 것이 맞다.(양방향으로 영향을 미쳐야 한다.)

## 1. Supervision Job

해당 job을 평범한 job과 동일하다. <span style = "background-color:#fff5b1">한 가지 차이점은 cancellation이 아래로만 전달된다는 점이다.</span>

```kotlin
val supervisor = SupervisorJob()
with(CoroutineScope(coroutineConbash + supervisor)) {
    // launch the first child -- its exception is ignored for this example (don't do this in practice!)
    val firstChild = launch(CoroutineExceptionHandler { _, _ ->  }) {
        println("The first child is failing")
        throw AssertionError("The first child is cancelled")
    }
    // launch the second child
    val secondChild = launch {
        firstChild.join()
        // Cancellation of the first child is not propagated to the second child
        println("The first child is cancelled: ${firstChild.isCancelled}, but the second one is still active")
        try {
            delay(Long.MAX_VALUE)
        } finally {
            // But cancellation of the supervisor is propagated
            println("The second child is cancelled because the supervisor was cancelled")
        }
    }
    // wait until the first child fails & completes
    firstChild.join()
    println("Cancelling the supervisor")
    supervisor.cancel()
    secondChild.join()
}
```

위 예시 코드를 보면 `SupervisorJob()` 을 통해 superbisor job을 생성해 CoroutineConbash로 넣어준다.

그렇다면 위 코드의 실행 결과는 어떻게 될까? 해당 코드를 이해한다면 supervisorJob 작동 방식에 대해 명확히 이해할 수 있다. 코드의 흐름에 따라 쭉 살펴보자.

1. firstCild에서 예외 발생
    
    - 원래라면 cancellation이 양방향으로 전파돼 secondChild와 부모 job까지 모두 취소가 되게 된다.
    
    - 다음과 같이 supervisorJob만 제거한다면 firstChild 예외에 의해 부모가 종료되는 것을 확인할 수 있다.
        ```kotlin
        with(CoroutineScope(coroutineConbash)) {
            // launch the first child -- its exception is ignored for this example (don't do this in practice!)
            val firstChild = launch(CoroutineExceptionHandler { _, _ ->  }) {
                println("The first child is failing")
                throw AssertionError("The first child is cancelled")
            }
            // launch the second child
            val secondChild = launch {
                firstChild.join()
                // Cancellation of the first child is not propagated to the second child
                println("The first child is cancelled: ${firstChild.isCancelled}, but the second one is still active")
                try {
                    delay(Long.MAX_VALUE)
                } finally {
                    // But cancellation of the supervisor is propagated
                    println("The second child is cancelled because the supervisor was cancelled")
                }
            }
            // wait until the first child fails & completes
            firstChild.join()
            println("Cancelling the supervisor")
        
        }
        ```
        ```bash
        The first child is failing
        Exception in thread "main" java.lang.AssertionError: The first child is cancelled
        at FileKt$main$1$1$firstChild$2.invokeSuspend (File.kt:9) 
        at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith (ContinuationImpl.kt:33) 
        at kotlinx.coroutines.DispatchedTask.run (DispatchedTask.kt:108) 
        ```
        
    - 하지만 여기서는 `supervisorJob`이 활용된다. 따라서 cancellation이 아래로만 전파되기 때문에 secondChild나 부모에 아무 영향을 미치지 않는다.
    
2. second child는 delay에 의해 코루틴이 일시 중지한다.
3. supervisor job이 cancel된다.
4. supervisor job이든 상관없이 부모 job이 취소가 되어 자식인 secondChild 역시 취소가 된다.

```bash
The first child is failing
The first child is cancelled: true, but the second one is still active
Cancelling the supervisor
The second child is cancelled because the supervisor was cancelled
```

## 2. Supervision scope
위에서는 supervisor job을 만들었다면 supervisor scope 역시 만들 수 있다. 

> Creates a [CoroutineScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) with [SupervisorJob](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html) and calls the specified suspend [block](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html) with this scope.
> 

즉 위와 같이 coroutineScope에 supervisor job을 지정하는 것과 동일한 작업을 진행할 수 있다.

```kotlin
try {
    supervisorScope {
        val child = launch {
            try {
                println("The child is sleeping")
                delay(Long.MAX_VALUE)
            } finally {
                println("The child is cancelled")
            }
        }
        // Give our child a chance to execute and print using yield 
        yield()
        println("Throwing an exception from the scope")
        throw AssertionError()
    }
} catch(e: AssertionError) {
    println("Caught an assertion error")
}
```

그렇다면 이 경우도 코드를 자세히 한번 살펴보자!

1. yield()에 의해 child가 먼저 실행 후 delay
2. supervisorScope에서 예외 발생, 자식인 child도 cancel
3. Scope에서 던진 AssertionError 예외 처리

## 3. exceptions in supervised coroutines

일반적인 job과 supervisor job의 또 다른 핵심적인 차이는 exception을 handling하는 방식이다.

supervisor job 내부에 존재하는 child들은 <span style = "background-color:#fff5b1">각자 handler를 등록해서 exception을 처리해야 한다.</span> 그 이유는 예외가 부모로 전달되지 않기 때문에 child의 내부에서 예외가 발생하고 처리가 되지 않는다면 이로 인해 프로그램이 바로 종료되기 때문이다.

```kotlin
val handler = CoroutineExceptionHandler { _, exception -> 
    println("CoroutineExceptionHandler got $exception") 
}
supervisorScope {
    val child = launch(handler) {
        println("The child throws an exception")
        throw AssertionError()
    }
    println("The scope is completing")
}
println("The scope is completed")
```

```bash
The scope is completing
The child throws an exception
CoroutineExceptionHandler got java.lang.AssertionError
The scope is completed
```

child에서 발생한 예외가 handler에 의해 처리가 되고 부모까지 잘 마무리된 것을 확인할 수 있다.

그렇다면 한번 child에 등록된 handler를 지워볼까?

```kotlin
supervisorScope {
    val child = launch() {
        println("The child throws an exception")
        throw AssertionError()
    }
    println("The scope is completing")
}
println("The scope is completed")
```

```shell
The scope is completing
The child throws an exception
Exception in thread "main @coroutine#2" java.lang.AssertionError
	at FileKt$main$1$1$child$1.invokeSuspend(File.kt:11)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	//.. 예외 코드
The scope is completed
```

왜 이런 결과가 나왔는지 생각해보자.

1. child에서 에러가 발생하고 이로 인해 child 코루틴은 cancellation이 일어난다. (exception이 처리가 되지 않아 콘솔창에 에러가 뜬다.)
2. supervisorScope이기 때문에 해당 cancellation은 위로 전파되지 않는다.
3. 성공적으로 프로그램이 마무리된다.

<br>

# 🔗 마무리하며

현재 우리 프로젝트에서는 corotuine이 다양한 방식으로 사용되고 있지는 않다. 단순히 서버에 값을 저장하거나 불러오기 위해 사용하고 있기 때문에 복잡한 로직이 존재하지는 않는다. 따라서 우선은 가장 기본적인 `CoroutineExceptionHandler`를 적용하기로 하였다. 추후 서비스를 계속해서 업데이트하면서 coroutine 사용이 복잡해진다면 에러 처리 역시 조금 더 세부적으로 진행해보고자 한다!

coroutine 공식 문서를 학습하고 정리하는 과정에서 내가 놓치고 있었던 부분이 정말 많다는 것을 다시 한번 느낄 수 있었다. 앞으로 coroutine 공식 문서 내용을 하나씩 다 학습하고 정리해보고자 한다! (코루틴 마스터가 되어야지!우하하!!)