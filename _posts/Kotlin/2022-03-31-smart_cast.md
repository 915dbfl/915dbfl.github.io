---
title: "[Kotlin] Smart Cast"
excerpt: "코틀린에서의 타입 자동 변환"
categories:
  - Kotlin
tag:
  - kotlin
  - smart_cast

last_modified_at: 2022-03-31
toc: true
toc_sticky: true
search: true
---

# 🙋‍♀️Smart Cast?

* 코틀린에서 컴파일러가 타입을 **자동**으로 변환해주는 것을 말한다.
  * 🙄 **명시적**으로 타입을 변환할 경우에는 **as, as?**를 사용한다.
  ```
  val tmp1: Int = b as Int
  ```
    * 변수 b를 Int 타입으로 변환해 a에 대입한다.
  
    * b가 Int 타입에 <u>적합하지 않을 경우</u>, **ClassCastException 예외**가 발생한다.

  * 🙄 **as?**를 사용하게 되면 b가 Int 타입에 적합하지 않을 경우, **null**이 a에 대입되게 된다.
  ```
  val tmp1: Int = b as? Int
  ```
  
<br>

* 스마트 캐스트가 진행되는 경우
  * **null 체크**를 할 경우
  * **is, !is** 연산자로 타입 확인을 할 경우

<br>

# 👩null check
```
val str: String? = "안녕하세요!"
if(str != null) print(str.length) #6
```
🙄 위의 예제처럼 str 변수가 null인지 체크한 후, <u>null이 아님이 확인되면</u> 자동으로 str을 **null이 불가능한 String 타입**으로 변환해 준다. (**스마트 캐스트**)

* str은 **if문 안에서만** null이 불가능한 String 타입을 갖게 된다.

<br>

# 👩is, !is
* is, !is도 null check와 마찬가지로 해당 타입일 경우, 스마트 캐스트가 일어난다.

```
fun main(){
  smartCast("안녕하세요!")
  smartCast(2521)
}

fun smartCast(e: Any){
  if (x is String) print(e.length) // x가 자동적으로 String으로 캐스팅된다.
  if (x is Int) print(e*10) // x가 자동적으로 Int로 캐스팅된다.
}
```

* 코틀린의 모든 클래스는 Any 클래스를 상속받는다.
* Any의 경우, null을 제외한 모든 타입의 값으로 지정할 수 있다.


🙄 위의 코드를 when을 이용하게 처리할 수 있다.

```
fun main(){
  smartCast("안녕하세요!")
  smartCast(2521)
}

fun smartCast(e: Any){
  when(e){
    is String -> print(e.length)
    is Int -> print(e*10)
  }
}
```

<br>

🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

## 📃참고
* [https://kotlinlang.org/docs/typecasts.html#unsafe-cast-operator](https://kotlinlang.org/docs/typecasts.html#unsafe-cast-operator)
* [https://hongku.tistory.com/352](https://hongku.tistory.com/352)
* [https://bbaktaeho-95.tistory.com/24](https://bbaktaeho-95.tistory.com/24)
* [https://bbaktaeho-95.tistory.com/21?category=782641](https://bbaktaeho-95.tistory.com/21?category=782641)