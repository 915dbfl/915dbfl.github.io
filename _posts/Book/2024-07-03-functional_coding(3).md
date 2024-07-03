---
title: "[개발 서적] 쏙쏙 들어오는 함수형 코딩(3)"
excerpt: "일급 값과 함수형 도구에 대해 알아보기"
categories:
  - Book
tag:
  - 개발 서적
  - 쏙쏙 들어오는 함수형 코딩
  - 일급 함수
  - 함수형 도구

last_modified_at: 2024-07-03
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며

지금까지 

1️⃣ 함수형 프로그래밍이란 무엇인지

2️⃣ `액션 / 데이터 / 계산`을 잘 활용할 수 있는 방법은 무엇인지

3️⃣ 계층형 설계를 적용해 유지보수 / 테스트 / 재사용 관점에서 용이한 함수는 어떻게 만들 수 있는지

에 대해 다뤄보았다. 이번 게시글에서는 함수형 프로그래밍하면 가장 많이 언급되는 `일급 추상 / 일급 함수`에 대해 다뤄보고자 한다.

<br>

# 🔗 일급?

`일급 추상`, `일급 함수`, `일급 값`

그렇다면 여기서 말하는 <span style = "background-color:#fff5b1">일급</span>이라는 것이 무엇일까?

> 일급
  - 언어에 있는 다른 값처럼 쓰일 수 있는 것을 의미한다.
  - <span style = "background-color:#fff5b1">변수나, 배열, 인자 등 언어 전체</span>에서 사용할 수 있다.

그렇다면 일급 값을 언제 어떻게 사용할 수 있는지 다음 예를 통해 조금 더 명확히 알아보자!

<br>

## 🔗 예제1) 함수 이름에 있는 암묵적 인자 일급 값으로 만들기

코드를 짜다 보면 <span style = "background-color:#fff5b1">내부 구현은 거의 동일하지만, 함수 이름이 구현의 차이를 만드는 경우</span>가 있다. 예를 들어 `setItemName`, `setItemPrice`는 내부적으로 아이템의 특정 요소의 값을 바꾸는 비슷한 구현을 가지고 있지만 함수 이름에서 알 수 있듯이 각각 이름과 가격을 변경한다. 이러한 함수들이 많아진다면 비슷한 코드 중복이 계속 생기게 된다.

이럴 경우, <span style = "background-color:#fff5b1">함수 이름에 있는 암묵적인 인자(Name, Price)를 명시적으로 드러낸다.</span>

```kotlin
fun setItemValue(bookCart: Array<Book>, field: String, value: String)
```

이렇게 위에서 `Name`, `Price`와 같이 함수 이름에 있던 암묵적인 인자를 `field`라는 명시적인 인자를 통해 전달해준다.

> ✏️ String으로 필드명을 전달하는 것은 좋을까?
  - 위와 같이 String으로 필드명을 전달하는 방식을 사용한다면 오타 등과 같은 오류로부터 안전하지 않다고 생각할 수 있다.
  - 이를 위해 `Enum`과 같이 `타입 시스템`을 활용한다면 보다 안전하게 필드명을 전달할 수 있다.

<br>

## 🔗 예제2) 일급이 아닌 문법을 일급으로 바꾸기

일급이란 <span style = "background-color:#fff5b1">언어 전체에서 사용할 수 있는 것</span>을 의미한다고 했다. 

언어 자체에서 제공하는 문법들은 일급이 아닌 경우가 많다. 그렇다면 이러한 문법을 일급으로 만드는 것은 가능할까? 그렇다! 많은 언어에서 `+` 연산자를 통해 `더하기 기능`을 제공한다. 그렇다면 이 연산자를 일급으로 만들어보자!

```kotlin
fun plus(num1: Int, num2: Int): Int {
  return num1 + num2
}
```

위에서 만든 함수는 더하기 기능을 제공하는 `일급 함수`이다. 따라서 인자, 리턴값 등 어디서나 활용이 될 수 있다.

<br>

## 🔗 예제3) 일급 함수를 활용해 콜백 함수 등록하기

예제1번에서 말했듯이 코드를 짜다보면 내부 구현이 거의 동일한 경우가 존재한다. 이번에는 이렇게 비슷한 내부 구현 내에서도 서로 다른 부분을 콜백으로 등록하는 방식에 대해 다뤄보고자 한다.

우선 코드를 살펴보기 전에 <span style = "background-color:#fff5b1">고차함수</span>에 짚고 넘어가자.

> 고차 함수
  - <span style = "background-color:#fff5b1">인자로 함수를 받거나 리턴값으로 함수를 리턴하는 함수</span>

그렇다면 이제 다음 코드를 확인해보자.

```kotlin

fun discountPrice(cart: Array<Cart>, amount: Double) {
  for (i in 0 until cart.size) { // 배열 속 모든 cart를 반복
    // discount 진행
  }
}

fun doublePrice(cart: Array<Cart>, amount: Double) {
  for (i in 0 until cart.size) { // 배열 속 모든 cart를 반복
    // double 진행
  }
}

```

위의 두 함수 모두 내부에서 `for`문을 통해 cart 배열 속 모든 요소를 확인하고 있다. 이렇게 특정 본문을 제외하고 동일한 구성을 가지는 경우, <span style = "background-color:#fff5b1">본문을 콜백으로 빼내어 리팩토링할 수 있다.</span>

```kotlin

// 일반적인 함수 이름 사용
fun modifyPrice(cart: Array<Cart>, modify: (Int) -> Int ) {
  for (i in 0 until cart.size) { // 배열 속 모든 cart를 반복
    modify(cart[i].price) // 콜백 호출
  }
}

// 원하는 콜백을 넣어줌
modifyPrice(cart, discount)
modifyPrice(cart, double)

```

위 코드를 본다면 <span style = "background-color:#fff5b1">서로 다른 본문을 콜백으로 빼내고, 필요한 본문을 콜백을 통해 넣어준다.</span> 이렇게 본문을 함수로 감싸 넣겨주면 감싼 함수는 호출되기 전까지 실행되지 않는다.

<br>

## 🔗 예제4) 함수를 리턴하는 고차함수 활용하기

`고차 함수`를 활용하면 불필요하게 반복되는 코드를 줄일 수 있다. 

`addItem(item)`, `removeItem(item)`을 진행할 때 에러가 발생하면 `logging`을 진행하는 코드를 짜려고 한다. 이를 위해서는 다음과 같이 `try-catch`을 반복해서 작성해야 한다.

```kotlin
try {
  addItem(item)
} catch(e: SomeException) {
  logError(e)
}

try {
  removeItem(item)
} catch(e: SomeException) {
  logError(e)
}
```

이러한 불필요하게 반복되는 코드를 줄이기 위해 다음과 같이 <span style = "background-color:#fff5b1">고차 함수</span>를 만들어 일반화해서 활용할 수 있다.

```kotlin
fun withLogging(func: Item -> Unit): (Item) -> Unit {
  return { item -> func(item) }
}

// 함수를 나타내는 변수이기 때문에 변수명은 동사형을 쓴다.
var addItemWithLogging = withLogging(addItem)
var removeItemWithLogging = withLogging(removeItem)
```

<br>

# 🔗 함수형 도구

해당 게시글에서 다룰 함수형 도구는 크게 <span style = "background-color:#fff5b1">map, filter, reduce</span> 세 가지이다. 이러한 함수형 도구들을 사용하면 <span style = "background-color:#fff5b1">배열에 대한 중복되는 반복문을 없앨 수 있다.</span> 여기에서는 각각의 쓰임새만 간단히 짚고 넘어가고, 자세한 사용법에 대해서는 다루지 않을 것이다. 이러한 map / filter / reduce 외에도 다양한 `함수형 도구`가 존재하기 때문에 관련된 내용은 언어 공식 홈페이지를 참고하길 바란다!

<br>

## 🔗 용도 및 내부 구현(kotlin)

우선 위 세가지 함수가 어떠한 용도로 사용되는지 먼저 알아보자!

- `map`
  - 배열 모든 항목에 함수를 적용해 <span style = "background-color:#fff5b1">새로운 배열을 리턴</span>한다.
  - 각 항목에 대해서 <span style = "background-color:#fff5b1">콜백 함수가 적용</span>된다.

- `filter`
  - 배열의 <span style = "background-color:#fff5b1">하위 집합을 선택해 새로운 배열을 리턴</span>한다.
  - `술어(true/false를 리턴하는 함수)`를 전달해 원하는 항목을 선택한다.
  - `조건문` 대신 활용할 수 있다.
  
- `reduce`
  - `초깃값`을 가지고 배열의 항목을 조합해 <spen style = "background-color:#fff5b1">하나의 결과값</span>을 만들어낸다.

그렇다면 `kotlin`에서 위 세가지 함수에 대한 내부 구현을 살펴보자!

```kotlin
inline fun <T, R> Array<out T>.map(
    transform: (T) -> R
): List<R>

inline fun <T> Array<out T>.filter(
    predicate: (T) -> Boolean
): List<T>

inline fun <S, T : S> Array<out T>.reduce(
    operation: (acc: S, T) -> S
): S
```

(위와 같이 Array 뿐만 아니라 다양한 배열을 대상으로 해당 함수들을 사용할 수 있다. 자세한 내용은 [공식 문서](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/)를 참고하길 바란다.) 위 코드와 같이 <span style = "background-color:#fff5b1">원하는 동작을 하는 콜백 함수만을 전달한다면</span> 배열 요소들에 대해 다양한 작업을 쉽고 간편하게 할 수 있다. 

map / filter의 경우 반환되는 값이 List라는 것을 확인할 수 있다. 이처럼 함수형 도구를 잘 활용한다면 <span style = "background-color:#fff5b1">여러 작업들을 chaining해 더욱 효율적으로 작업할 수 있다.</span>

```kotlin
// ex)
numArray.map {it * 2}.filter {it % 2 == 0}
```

> ✏️ 함수형 도구를 통해 <span style = "background-color:#fff5b1">반복하는 일과 반복문에서 하려는 일을 분리할 수 있다.</span> 위에서는 배열에 대한 함수형 도구만을 다뤘다. 하지만 함수형 도구는 배열에만 국한된 것이 아니라 객체나 다른 값에 대해서도 직접 구현해 사용할 수 있다. 

<br>

# 🔗 마무리하며

함수형 프로그래밍하면 빠질 수 없는 `일급 함수 및 고차 함수`에 대해 다뤄보았다. 또한, 배열을 다룰 때 정말 많이 활용되는 `함수형 도구`에 대해서도 알아보았다. 

코드를 짜며 다른 함수를 인자로 받는 함수는 어느 정도 활용해보았지만 함수를 리턴하는 고차함수는 많이 활용해본 적이 없는 것 같다. 나는 주로 함수를 코드 가독성을 위한 분리 도구로 사용했던 것 같다. 이번 내용을 학습하면서 함수형 프로그래밍에서 함수를 잘 활용하는 것이 어떤 것인지를 학습할 수 있었다. 

함수형 프로그래밍... 배우면 배울수록 참 흥미롭다..✨