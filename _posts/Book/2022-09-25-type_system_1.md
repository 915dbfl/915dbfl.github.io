---
title: "[Kotlin in action] 6장: 코틀린 타입 시스템(1)"
excerpt: ""
categories:
  - Book
tag:
  - kotlin type system
  - kotlin in action

last_modified_at: 2022-09-25
toc: true
toc_sticky: true
search: true
---

# 👩 타입 시스템

* 코틀린의 타입 시스템은 **코드의 가독성을 향상**시키는 데 도움을 준다.
* 종류
  * Nullable type
  * 읽기 전용 컬렉션

<br>

## 🙋‍♀️ nullable type

> 코틀린을 비롯한 최신 언어에서 null에 대한 접근 방법은 가능한 한 이 문제를 실행 시점에서 🔔컴파일🔔 시점으로 옮기는 것이다. 

> 널이 될 수 있는지 여부를 타입 시스템에 추가함으로써 컴파일러가 여러 가지 오류를 컴파일 시 미리 감지해서 실행 시점에 발생할 수 있는 예외의 가능성을 줄일 수 있다.

* 널이 될 수 있는 타입과 널이 될 수 없는 타입을 구분하면 **각 타입의 값에 어떤 연산이 가능할지** 명확히 이해할 수 있다.
* 즉, 실행 시점에 예외를 발생시킬 수 있는 연산을 판단할 수 있다.

<br>

### ➡️ ?
* **nullable type**을 표시하기 위해서는 타입 이름 뒤에 **?**를 명시한다.

<br>

### ➡️ 안전한 호출 연산자: ?.
* 안전한 호출의 결과 타입도 **nullable type**이라는 사실에 유의해야 한다.

<br>

### ➡️ 엘비스 연산자: ?:
* null 대산 사용할 **디폴트 값을 지정**힐 때 사용하는 연산자이다.

```kotlin
  fun Person.countryName() = company?.address?.country ?: "Unknown"
```
* `company?.address?.country?` 값이 null이 아닐 경우, 해당 값이 함수의 결과가 되지만, null일 경우에는 `Unkown`이 결과가 된다.

<br>

```kotlin
  val address = person.company?.addres ?: throw IllegalArgumentException("No address")
```
* 코틀린에서는 return, throw도 식에 해당한다.
* 위와 같이 엘비스 연산자의 우항에 `return`, `throw` 등의 연산을 넣어 더욱 편리하게 사용할 수 있다.

<br>

### ➡️ 안전한 캐스트: as?
* 대상 값을 as로 지정한 타입으로 바꿀 수 없으면 `ClassCastException`이 발생한다.
* `as?`는 값을 대상 타입으로 변활할 수 없으면 **null**을 반환한다.
* 일반적인 패턴은 **캐스트를 수행한 뒤**에 **엘비스 연산자**를 사용하는 것이다.

```kotlin
  val otherPerson = o as? Person ?: return false
```

<br>

### ➡️ 널 아님 단언: !!

* `!!`는 어떤 값이든 **널이 될 수 없는 타입으로 바꿀 수 있다.**

```kotlin
  fun ignoreNulls(s: String?) {
    val sNotNull: String = s!! <-🔔예외가 가리키는 곳!
    println(sNotNull.length)
  }

>>> igNoreNulls(null)  
```

위의 경우, 널이 아님을 단언했으나 널을 만났으므로 NPE가 발생한다. 하지만 여기서 주목해야 할 점은 그 예외가 **null을 사용하는 코드**가 아니라 **단언문이 위치한 곳**을 가리킨다는 점을 유의해야 한다.

<br>

그렇다면 `!!`는 어떤 때 사용하는 것이 좋을까?

* 어떤 함수가 값이 널인지 검사한 다음 다른 함수를 호출한다.
  * 이 경우, 컴파일러는 호출된 함수 안에서 <u>안전하게 그 값을 사용할 수 있음을 인식할 수 없다!<u/>
* 다른 함수를 호출할 때 `!!`을 통해 널이 아닌 값을 전달한다면 굳이 널 검사를 또 하지 않아도 된다.

<br>

`!!`를 사용할 때 주의해야 할 점도 있다!🔔

* `!!`를 널에 대해 사용하면 **스택 트레이스(stack trace)** 예외가 발생한다.
* 이 예외에는 예외가 발생한 **파일의 줄**에 대한 정보는 들어있지만 **어떤 식**에서 발생했는지는 정보가 없다.
* 🔔 따라서 `!!`를 한 줄에 연속해서 쓰는 일은 피하자!!
  ```kotlin
    person.company!!.address!!.country //-> company, address 중 어디에서 예외가 발생했는지 알 수 없다!
  ```

<br>

### ➕ 자바에서의 null 처리
* 자바에서는 널이 될 수 있는지 여부를 애노테이션을 통해 하기도 한다.
  * @Nullable / @NotNull

<BR>

## 🙋‍♀️ 라이브러리 함수: let
* 널이 **될 수 있는 값**을 널이 **아닌** 값만 인자로 받는 함수에 넘기는 경우이다.
  * 널이 **될 수 있는 값**에 대해 **안전한 호출 구문**을 사용한다.
  * let을 호출한다.
  * 널이 될 수 **없는** 타입을 인자로 받는 **람다**를 let에 전달한다.
* 아주 긴 식이 있고, 그 값이 **널이 아닐 때** 수행해야 하는 로직이 있을 때 let을 쓰면 편리하다.

  ```kotlin
    email?.let{
      sendEmailTo(it)
    }
  ```
  * email이 널이 아니라면 it은 람다 안에서 널이 아니다.
  * email이 널이라면 <u>아무 일도 일어나지 않는다.</u>

  
  ➡️ 그렇다면 <u> 안전한 호출을 사용하지 않으면</u> 어떻게 될까?

  ```kotlin
    person.let {sendEmailTo(it)}
  ```
  * let 함수에 넘겨지는 람다 속 **it은 널이 될 수 있는 타입**으로 취급된다.

  <br>

  ## 🙋‍♀️ lateinit: 나중에 초기화

  * 코틀린은 일반적으로 **생성자**에서 모든 프로퍼티를 **초기화**해야 한다.

  ➡️ 다음의 경우를 한 번 살펴보자!

  ```kotlin
    class MyService{
      fun performAction(): String = "foo"
    }

    class MyTest{
      private var myService: MyService? = null // 생성자에서 초기화하기 위해 널이 될 수 있는 타임으로 선언 후, null로 초기화한다.

      @Before fun setUp(){
        myService = MyService() // 진짜 초기값을 지정
      }
    }
  ```
  위에서 볼 수 있듯이 생성자에서 초기화를 진행해야 하기 때문에 null을 통해 초기화를 우선 지정하는 경우가 많다. 하지만 이러한 코드는 보기에 안좋기 때문에 이를 해결하기 위해 사용하는 것이 **lateinit** 변경자이다.
  * lateinit 프로퍼티는 **var**이어야 한다.
    * **val**의 경우, <u>final 필드로 컴파일</u>되기 때문에 반드시 **생성자 안**에서 초기화해야 한다.

  ```kotlin
  class MyTest{
    private lateinit var myService: Myservice // 초기화하지 않고 널이 될 수 없는 프로퍼티를 선언!

    @Before fun setUp(){
      myService = MyService()
    }

    @Test fun testAction(){
      Assert.assertEquals("foo",
        myService.performAction()) // 널 검사 수행하지 않고 프로퍼티 사용!
    }
  }
  ```

  <br>

  ## 🙋‍♀️ ?가 없어도 널이 될 수 있는 타입이 아니다!

  * 널이 될 수 있는 타입의 확장 함수로 `isNullOrEmpty`, `isNullOrBlack` 메서드가 있다.

  ```kotlin
    fun verifyUserInput(input: String?){
      if(input.isNullOrBlank()){ // 안전한 호출을 하지 않아도 된다.
        println("...")
      }
    }
  ```
  그렇다면 String의 isNullOrBlank 확장 함수를 자세히 살펴보자!

  ```kotlin 
    fun String?.isNullOrBlank(): Boolean =
      this == null || this.isBlank()
  ```
  * 자바의 경우, 메서드 안 this는 메서드가 호출된 수신 객체를 가리키므로 항상 **널이 아니다.**
  * 코틀린에서는 **널이 될 수 있는 타입의 확장 함수 안**에서는 this가 널이 될 수 있다.

  <br>

## 🙋‍♀️ 타입 파라미터의 널 가능성

```kotlin
fun <T> printHashCode(t: T){
  println(t?.hashCode())
}
```
🙂 위 제네릭 함수의 정의를 보다 보면 다음의 궁금증이 생길 수 있다! `타입 파라미터 T는 ?가 없는데 t에 안전한 호출을 사용했네?`

* 코틀린에서는 함수나 클래스의 모든 `타입 파라미터`는 기본적으로 **널이 될 수 있다.**
* 위 T의 경우 `Any?` 타입이 된다.

🙄 만약 타입 파라미터가 확실이 널이 아님을 단언하고 싶다면 타입 상한을 지정해야 한다.

```kotlin
fun <T: Any> printHashCode(t: T){
  println(t.hashCode())
}
```

* 타입 파라미터는 널이 될 수 있는 타입을 표시할 때 ?를 사용해야 한다는 규칙의 유일한 예외이다.

<br>

## 🙋‍♀️ 널 가능성과 자바

### ➡️ 애노테이션이 없는 자바 타입은 코틀린의 플랫폼 타입!

* 자바에서는 **애노테이션**을 통해 **널 가능성 정보**를 표시한다.
  * `@Nullable String` = 코틀린의 `String?`

하지만 애노테이션이 없는 경우는? 그런 경우, 자바의 타입은 코틀린의 **플랫폼 타입**이 된다.

<br>

### ➡️ 플랫폼 타입?
> 코틀린이 **널 관련 정보를 알 수 없는** 타입

* 따라서 타입을 널이 될 수 **있는** 타입으로 처리해도 되고, 널이 될 수 **없는** 타입으로 처리해도 된다.

<br>

### ➡️ 상속

자바 메서드를 코틀린에서 오버라이드 할 경우가 있다! 이떄 고민해야 할 부분은 **메서드의 파라미터와 반환 타입**을 **널이 될 수 있는 타입으로 선언할지 말지** 결정해야 한다.

결론을 말하자면 **둘 다 가능하다!** 따라서 자바 클래스나 인터페이스를 코틀린에서 구현할 경우, <u>널 가능성을 제대로 처리</u>해야 하는 일이 중요하다!

<br>

## 🙋‍♀️ 코틀린의 원시 타입

먼저 본격적인 내용에 들어가기 전에 `원시 타입/참조 타입/래퍼 타입`에 대해 알아보고 넘어가자!!

> 원시 타입(primitive type): 변수에 값이 **직접** 들어가는 타입

* ex) int, byte, long...
* 원시 타입에 대해서 **메서드를 호출**하거나, **컬렉션**에 원시 타입을 담을 수 없다.

> 참조 타입(reference type): 변수에 **메모리상의 객체 위치**가 들어가는 타입

* ex) String...

> 래퍼 타입: Integer, Double의 경우

<br>

### ➡️ 코틀린: 원시 타입 vs 래퍼 타입
* 자바에서는 **참조 타입**이 필요한 경우, 원시 타입을 감싼 **래퍼 타입**을 사용한다.
  * `Collection<int>`가 아닌 `Collection<Integer>`

* 코틀린의 경우, 래퍼 타입과 원시 타입을 **따로 구분하지 않는다.**
  * 즉, 원시 타입에 대해 **메서드 호출이 가능**하고, **컬렉션**에 원시 타입을 담을 수 **있다.**

* 단, 원시 타입과 래퍼 타입이 따로 구분되지 않는다고 해서 **항상 객체로 표현되는 것은 아니다.**

<br>

### ➡️ 자바: 널이 될 수 있는 원시 타입 처리 => 래퍼 타입

* **null 참조**를 자바에서는 **참조 타입** 변수에만 대입할 수 있다.
* 따라서 코틀린에서 **널이 될 수 있는 원시 타입**을 사용하면 자바에서는 **래퍼 타입**으로 컴파일된다.

<br>

### ➡️ 제네릭 클래스는 무조건 래퍼 타입!

* 자바 가상머신(JVM)의 경우, **타입 인자**로 **원시 타입을 허용하지 않는다.**
* 자바나 코틀린 모두 제네릭 클래스는 항상 **래퍼 타입**을 사용해야 한다.

<BR>

## 🙋‍♀️ 코틀린에서의 숫자 변환

* 코틀린은 숫자 타입을 **자동 변환하지 않는다.**
* 따라서 타입을 명시적으로 변환해서 같은 타입의 값을 만들어야 한다.
  * `toInt()`, `toByte()`, `toLong()`...
  * 범위가 더 넓은 값에 대입(int -> long)을 해도 자동 변환되지 않는다.

<br>

## 🙋‍♀️ 코틀린의 특별한 타입

### ➡️ Any, Any?: 최상위 타입
* **자바**에서의 최상위 타입은 `Object`이다.(원시타입 제외)
* **코틀린**에서는 `Any`가 모든 **널이 될 수 없는 타입**의 조상 타입이다.(원시타입 포함)
  * 널을 **포함**하는 모든 값을 대입할 수 있는 변수를 선언하기 위해서는 `Any?` 타입을 사용해야 한다.
* 자바에서의 **Object 타입**은 코틀린에서 **Any!(널의 여부를 알 수 없으므로 플랫폼 타입)**으로 취급된다.
  * Object에 있는 메서드를 Any에서 사용하기 위해서는 Object로 캐스트해야 한다.

<br>

### ➡️ Unit 타입: 코틀린에서의 void

> 자바 `void` = 코틀린 `Unit`

* void와는 다른 Unit의 특징
  * 모든 기능을 갖는 **일반적인 타입**이다.
    * Unit 타입에 Unit이라는 단 하나의 값이 존재한다.
  * Unit을 타입 인자로 쓸 수 있다.
  
  ```kotlin
  interface Processor<T> {
    fun process(): T
  }

  class NoResultProcessor: Processor<Unit> { // Unit을 반환하지만 타입을 지정할 필요 없다.
    override fun process(){
      //여기서 return을 명시할 필요 없다.
    }
  }
  ```
  * **제네릭 파라미터**를 반환하는 함수를 **오버라이드**할 경우, 반환 타입으로 `Unit`을 쓰기 유용하다.
  * 컴파일러가 묵시적으로 `return Unit`을 넣어주기 때문에 return을 명시하지 않아도 된다.

<br>

### ➡️ Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다.

* 특정 **반환 값이 없고**, **정상적으로 끝나지 않을 때** `Nothing` 타입을 반환한다.
  ```kotlin
  fun fail(message: String): Nothing{ // 예외를 던져 함수가 정상적으로 끝나지 않는다.
    throw IllegalStateException(message)
  }

  >>>fail("Error occurred")
  java.lang.IllegalStateException: Error occurred
  ```
* <u>아무 값도 포함하고 있지 않아</u>, 함수의 **반환 타입**이나 반환 타입으로 쓰일 **타입 파라미터**로만 쓸 수 있다.
* 엘비스 연산자의 우항에 사용해서 전제 조건을 검사할 수 있다.
  ```kotlin
  val address = company.address ?: fail("No address")
  println(address.city)
  ```

  <br>

🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

## 📃참고
* 'Kotlin in action': 6장(코틀린 타입 시스템)