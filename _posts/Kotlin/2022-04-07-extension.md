---
title: "[Kotlin] Extensions"
categories:
  - Kotlin
tag:
  - kotlin
  - extensions

last_modified_at: 2022-04-07
toc: true
toc_sticky: true
search: true
---

<br>

# 🙋‍♀️Extensions?

* extension(확장)을 통해서 간단하게 객체의 함수, 프로퍼티를 확장 정의할 수 있다.

<br>

# 🙋‍♀️Extension functions
* index1, index2인 두 값을 교환하는 프로그램을 작성하고자 한다.

```
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```
* 확장 함수의 **this**는 **MutableLis**(receiver object)와 대응하게 된다.

* swap()은 **모든 MutableList**에서 사용할 수 있다.

* **generic**으로도 사용 가능하다.
  ```
  fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
  ```

🙄사용의 예를 한 번 살펴보자.
```
val list = mutableListOf(1, 2, 3)
list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'list'

print(list) // [3, 2, 1]
```

🙄 위의 예에서 확인했듯이 마치 <u>MutableList의 함수인 것처럼 swap()함수를 정의할 수 있다.</u>

<br>

# 🙋‍♀️Extension properties
extension 함수와 같이 extension properties도 제공한다.

```
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

<br>

# 🙇‍♀️Nullable receiver 사용!
* extension은 **nullable receiver 타입**과 함께 정의될 수 있다.
* 즉, null에 의해서도 extension이 호출될 수 있다는 의미이다.

```
fun Any?.toString(): String {
    if (this == null) return "null"
    // after the null check, 'this' is autocast to a non-null type, so the toString() below
    // resolves to the member function of the Any class
    return toString()
}
```
* extension 함수 안에서 null 체크가 이루어진다.
* 따라서 별도의 null 체크없이 extension 함수인 toString()을 사용할 수 있다.

<br>

# 🙋‍♀️Extension 사용 규칙

## 🙇‍♀️Extension은 **정적**으로 처리된다.

```
open class Shape
class Rectangle: Shape()

fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    println(s.getName())
}

printClassName(Rectangle())
```
* 위 예제를 확인하면 최종적으로 "Rectangle"이 출력될 것이라 생각할 것이다.
* 하지만 Extension의 경우 **정적**으로 처리되기 때문에 <u>실제로 참조되고 있는 타입인 Rectangle이 아니라</u> PrintClassName에 **선언된 타입인 Shape의 getName**이 호출되게 된다.

<br>

## 🙇‍♀️멤버가 Extension보다 우선이다.
```
class Person {
    fun getName() { println("yuri") }
}
fun Person.getName() { println("Hello, I'm yuri!") }

fun main() {
    Person().getName() //yuri
}
```
🙄 위 코드를 통해서 쉽게 이해할 수 있을 것이다.

* getName 함수는 멤버로도 선언되어 있으며, extension 함수로도 선언되어 있다.
* 이럴경우, 멤버가 우선이 된다.

<br>

## 🙇‍♀️ extesions의 범위
* extensions을 **선택적으로 import**해 사용할 수 있다.

```
package org.example.declarations

fun List<String>.getLongestString() { /*...*/}
```

```
package org.example.usage

import org.example.declarations.getLongestString

fun main() {
    val list = listOf("red", "green", "blue")
    list.getLongestString()
}
```
* extensions을 클래스 멤버로 선언할 경우에는 해당 클래스 내로 범위가 결정된다.

<br>

🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

## 📃참고
* <https://thdev.tech/kotlin/2020/10/27/kotlin_effective_08/>
* <https://medium.com/til-kotlin-ko/kotlin%EC%9D%98-extension%EC%9D%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94%EA%B0%80-part-1-7badafa7524a>
* <https://codechacha.com/ko/kotlin-extension-functions/>
* <https://kotlinlang.org/docs/extensions.html#declaring-extensions-as-members>
* <https://kotlinlang.org/docs/generics.html#variance>