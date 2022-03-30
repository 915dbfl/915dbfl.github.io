---
title: "[Kotlin] Data classes"
excerpt: "데이터와 관련된 반복된 코드를 없애는 효과적인 방법!"
categories:
  - Kotlin
tag:
  - kotlin
  - data_class

last_modified_at: 2022-03-30
toc: true
toc_sticky: true
search: true
---

# 🙋‍♀️Data Class?

자바 코드로 이름, 나이를 갖는 User 클래스를 만들어보자.

```
public class User {
    private final String name;
    private final int age;

    public User(String name, int age) {
      super();
      this.name = name;
      this.age = age;
    }
    
   public final String getName() {
      return this.name;
   }

   public final Int getAge() {
      return this.age;
   }

   //이하 생략
}
```

위 코드를 보면 알 수 있듯이, getter/setter와 같은 코드가 **반복**적으로 작성된다는 것을 확인할 수 있을 것이다. 이처럼 반복적인 코드를 '보일러 플레이트 코드'라고 한다. 이를 없애기 위해 코틀린에서 제공하는 개념이 바로 **'Data Class'**이다!

## 👩사용법

```
data class User(val name: String, val age: Int)
```
* class 앞 **'data'** 키워드를 추가해 생성자에 parameter들을 정의해줌으로써 간단하게 생성 가능하다.
* 파라미터가 val일 경우 getter만 생성되며, var의 경우 getter/setter가 자동으로 생성된다.
* data class를 생성하면 자동적으로 **equals, hashCode, getter, setter, toString**와 같이 java Object 클래스에 있는 기본 메소드 뿐만 아니라 추가적으로 **copy, componentN**이 존재하게 된다.


<br>

## 👩사용 시 주의사항
* **primary constructor**가 필수적으로 필요하다.
  * **한 개 이상**의 파라미터가 존재해야 한다.
* <u>primary constructor</u>의 파라미터들은 **val** 또는 **var**을 가져야 한다.
* data class는 abstract, open, sealed, inner 키워드를 가지지 **못한다.**

<BR>

## 👩Canonical Methods: toString, equals, hashCode
🙄 코틀린에서 모든 객체의 조상은 Any 객체이다. 여기서 Any에 선언된 메소드가 바로 Canonical Method이다. 따라서 코틀린의 모든 객체가 가지고 있는 메소드인 것이다.


* **toString() : String** : 클래스에 포함되어 있는 **데이터의 값들을 하나의 문자열**로 얻을 수 있다. 자동으로 생성되는 것으로 데이터를 추가 정의하거나 삭제한다고 해서 재정의 할 필요가 없다.
* **equals(other: Any?) : Boolean** : data class 속 데이터 값들의 일치를 비교한다. 모두 동일할 경우 true를 반환한다.
* **hashCode() : Int** : equlas로 동일한 두 객체는 동일한 hashCode가 리턴된다. 

<br>

## 👩copy
* **deep copy**를 지원하는 메소드이다.
* 단순한 복사 뿐만 아니라 **특정 property 값을 바꿀 수 있다.**

```
val alice = User("Alice", 20)
val bob = alice.copy(name = "bob") //이름만을 바꾸고 클래스 복사!
```
<br>

## 👩Destructuring & componentN
* 하나의 객체 속 내부 변수들을 **각각** 받을 수 있는 것을 말한다.
* 필요하지 않는 값은 **'_'**로 표시할 수 있다.

```
val alice: User("Alice", 20)
val (name, age) = alice
```

<br>

🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

## 📃참고
* [https://kotlinlang.org/docs/data-classes.html](https://kotlinlang.org/docs/data-classes.html)
* [https://readystory.tistory.com/85](https://readystory.tistory.com/85)
* [https://sabarada.tistory.com/197](https://sabarada.tistory.com/197)
* [https://thdev.tech/kotlin/2020/09/15/kotlin_effective_02/](https://thdev.tech/kotlin/2020/09/15/kotlin_effective_02/)