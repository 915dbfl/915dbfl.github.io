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

# ๐โโ๏ธExtensions?

* extension(ํ์ฅ)์ ํตํด์ ๊ฐ๋จํ๊ฒ ๊ฐ์ฒด์ ํจ์, ํ๋กํผํฐ๋ฅผ ํ์ฅ ์ ์ํ  ์ ์๋ค.

<br>

# ๐โโ๏ธExtension functions
* index1, index2์ธ ๋ ๊ฐ์ ๊ตํํ๋ ํ๋ก๊ทธ๋จ์ ์์ฑํ๊ณ ์ ํ๋ค.

```
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```
* ํ์ฅ ํจ์์ **this**๋ **MutableLis**(receiver object)์ ๋์ํ๊ฒ ๋๋ค.

* swap()์ **๋ชจ๋  MutableList**์์ ์ฌ์ฉํ  ์ ์๋ค.

* **generic**์ผ๋ก๋ ์ฌ์ฉ ๊ฐ๋ฅํ๋ค.
  ```
  fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
  ```

๐์ฌ์ฉ์ ์๋ฅผ ํ ๋ฒ ์ดํด๋ณด์.
```
val list = mutableListOf(1, 2, 3)
list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'list'

print(list) // [3, 2, 1]
```

๐ ์์ ์์์ ํ์ธํ๋ฏ์ด ๋ง์น <u>MutableList์ ํจ์์ธ ๊ฒ์ฒ๋ผ swap()ํจ์๋ฅผ ์ ์ํ  ์ ์๋ค.</u>

<br>

# ๐โโ๏ธExtension properties
extension ํจ์์ ๊ฐ์ด extension properties๋ ์ ๊ณตํ๋ค.

```
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

<br>

# ๐โโ๏ธNullable receiver ์ฌ์ฉ!
* extension์ **nullable receiver ํ์**๊ณผ ํจ๊ป ์ ์๋  ์ ์๋ค.
* ์ฆ, null์ ์ํด์๋ extension์ด ํธ์ถ๋  ์ ์๋ค๋ ์๋ฏธ์ด๋ค.

```
fun Any?.toString(): String {
    if (this == null) return "null"
    // after the null check, 'this' is autocast to a non-null type, so the toString() below
    // resolves to the member function of the Any class
    return toString()
}
```
* extension ํจ์ ์์์ null ์ฒดํฌ๊ฐ ์ด๋ฃจ์ด์ง๋ค.
* ๋ฐ๋ผ์ ๋ณ๋์ null ์ฒดํฌ์์ด extension ํจ์์ธ toString()์ ์ฌ์ฉํ  ์ ์๋ค.

<br>

# ๐โโ๏ธExtension ์ฌ์ฉ ๊ท์น

## ๐โโ๏ธExtension์ **์ ์ **์ผ๋ก ์ฒ๋ฆฌ๋๋ค.

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
* ์ ์์ ๋ฅผ ํ์ธํ๋ฉด ์ต์ข์ ์ผ๋ก "Rectangle"์ด ์ถ๋ ฅ๋  ๊ฒ์ด๋ผ ์๊ฐํ  ๊ฒ์ด๋ค.
* ํ์ง๋ง Extension์ ๊ฒฝ์ฐ **์ ์ **์ผ๋ก ์ฒ๋ฆฌ๋๊ธฐ ๋๋ฌธ์ <u>์ค์ ๋ก ์ฐธ์กฐ๋๊ณ  ์๋ ํ์์ธ Rectangle์ด ์๋๋ผ</u> PrintClassName์ **์ ์ธ๋ ํ์์ธ Shape์ getName**์ด ํธ์ถ๋๊ฒ ๋๋ค.

<br>

## ๐โโ๏ธ๋ฉค๋ฒ๊ฐ Extension๋ณด๋ค ์ฐ์ ์ด๋ค.
```
class Person {
    fun getName() { println("yuri") }
}
fun Person.getName() { println("Hello, I'm yuri!") }

fun main() {
    Person().getName() //yuri
}
```
๐ ์ ์ฝ๋๋ฅผ ํตํด์ ์ฝ๊ฒ ์ดํดํ  ์ ์์ ๊ฒ์ด๋ค.

* getName ํจ์๋ ๋ฉค๋ฒ๋ก๋ ์ ์ธ๋์ด ์์ผ๋ฉฐ, extension ํจ์๋ก๋ ์ ์ธ๋์ด ์๋ค.
* ์ด๋ด๊ฒฝ์ฐ, ๋ฉค๋ฒ๊ฐ ์ฐ์ ์ด ๋๋ค.

<br>

## ๐โโ๏ธ extesions์ ๋ฒ์
* extensions์ **์ ํ์ ์ผ๋ก import**ํด ์ฌ์ฉํ  ์ ์๋ค.

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
* extensions์ ํด๋์ค ๋ฉค๋ฒ๋ก ์ ์ธํ  ๊ฒฝ์ฐ์๋ ํด๋น ํด๋์ค ๋ด๋ก ๋ฒ์๊ฐ ๊ฒฐ์ ๋๋ค.

<br>

# ๐์ฐ์ฐ์ ์ค๋ฒ๋ก๋ฉ

์ด๋ฒ ์๊ฐ์ ๋ฐฐ์ด extension์ ์ด์ฉํด์ ์ฐ์ฐ์ ์ค๋ฒ๋ก๋ฉ๋ ์งํํ  ์ ์๋ค.
๊ฐ๋จํ ๋ด์ฉ์ด๊ธฐ์ ์ด๋ฒ ๊ฒ์๊ธ์์ ํจ๊ป ์ง๊ณ  ๋์ด๊ฐ๊ณ ์ ํ๋ค.

์ฐ์  **์ค๋ฒ๋ก๋ฉ์ด ๊ฐ๋ฅํ ์ฐ์ ์ฐ์ฐ์**๋ฅผ ๋ค์์ ํ๋ฅผ ํตํด์ ์ดํ๋ณด์.

|expression|translated to|
|----------|-------------|
|+a|a.unaryPlus()|
|-a|a.unaryMinus()|
|!a|a.not()|
|a++|a.inc()|
|a--|a.dec()|
|a+b|a.plus(b)|
|a-b|a.minus(b)|
|a*b|a.times(b)|
|a/b|a.div(b)|
|a%b|a.rem(b)|
|a..b|a.rangeTo(b)|
|a in b|b.contains(a)|
|a !in b|!b.contains(a)|
|a[i]|a.get(i)|
|a()|a.invoke()|
|a += b|a.plusAssign(b)|
|a -= b|a.minusAssign(b)|
|a *= b|a.timesAssign(b)|
|a /= b|a.divAssign(b)|
|a %= b|a.remAssign(b)|
|a > b|a.compareTo(b) > 0|
|a < b|a.compareTo(b) < 0|
|a >= b|a.compareTo(b) >= 0|
|a <= b|a.compareTo(b) <= 0|

๐ ๋ง์ ๋ณด์ฌ๋ ๋น์ทํ ๋ถ๋ถ์ด ๋ง์ ์ฝ๊ฒ ์ตํ ์ ์์ ๊ฒ์ด๋ค!

<br>

๊ทธ๋ ๋ค๋ฉด ์ ํ์ ์กด์ฌํ๋ ์ฐ์ ์ฐ์ฐ์๋ฅผ ์ด๋ป๊ฒ ์ค๋ฒ๋ก๋ฉํ๋์ง ํ ๋ฒ ์ดํด๋ณด์!
์๋ก Point ๊ฐ์ฒด๋ฅผ -Point ์ฐ์ฐ์ด ๊ฐ๋ฅํ๋๋ก ํ๊ณ ์ ํ๋ค.
```
data class Point(val x: Int, val y: Int)
```

* **-Point ์ฐ์ฐ**์ **unaryMinus()**๋ฅผ ์ค๋ฒ๋ก๋ฉ ํจ์ผ๋ก์จ ๊ตฌํํ  ์ ์๋ค.
  ```
  operator fun Point.unaryMinus() = Point(-x, -y)
  ```
  * ์ฐ์ฐ์ ์ค๋ฒ๋ก๋ฉ์ ์งํํ  ๊ฒฝ์ฐ, **operator ํค์๋**๋ฅผ ์ฌ์ฉํ๋ค.
  * Point ๊ฐ์ฒด์ extension function์ธ **unaryMinus()**๋ฅผ ์ ์ํ๋ค.

๐๊ฐ๋จํ์ง ์์๊ฐ? ๊ทธ๋ ๋ค๋ฉด ์ฌ์ฉ๋ฒ์ ํ ๋ฒ ์ดํด๋ณด์!

```
val point = Point(10, 20)

fun main() {
   println(-point)// "Point(x=-10, y=-20)"
}
```

<br>

๐โโ๏ธ ๋ถ์กฑํ ๋ถ๋ถ์ด ์๋ค๋ฉด ๋ง์ํด์ฃผ์ธ์! ๊ฐ์ฌํฉ๋๋ค!

## ๐์ฐธ๊ณ 
* <https://thdev.tech/kotlin/2020/10/27/kotlin_effective_08/>
* <https://medium.com/til-kotlin-ko/kotlin%EC%9D%98-extension%EC%9D%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94%EA%B0%80-part-1-7badafa7524a>
* <https://codechacha.com/ko/kotlin-extension-functions/>
* <https://kotlinlang.org/docs/extensions.html#declaring-extensions-as-members>
* <https://kotlinlang.org/docs/generics.html#variance>
* <https://kotlinlang.org/docs/operator-overloading.html#increments-and-decrements>