---
title: "[Delegate Pattern] kotlin delegated property로 보는 delegate pattern"
excerpt: "상속 대신 위임 패턴을 써볼까~?"
categories:
  - Android
tag:
  - kotlin
  - delegated property
  - delegate pattern

last_modified_at: 2025-01-13
toc: true
toc_sticky: true
search: true
---

## 🔗 들어가기 전
property를 만들 때 특정 로직이 필요하다고 하자. 이때 property를 필요할 때마다 선언하게 된다면 해당 특정 로직이 여러 곳에서 반복해서 사용되게 된다.

예를 들어 우리가 `위임 패턴이 적용되어 있다`고 알고 있는 `by lazy{}` 를 생각해보자. 지연 초기화가 해주는 작업은 뭘까? 말 그대로 선언 시점이 아닌 property가 처음 사용될 때 초기화가 될 수 있도록 해주며, 한번 초기화가 이루어진 이후에는 저장된 값을 가져와 사용할 수 있게 해준다.

> 여기서 `선언 시점이 아닌 property가 처음 사용될 때 초기화가 될 수 있도록 해주며, 한번 초기화가 이루어진 이후에는 저장된 값을 가져와 사용할 수 있게 된다.` 이 작업을 A작업이라고 해두자.

그렇다면 지연 초기화를 지원해주는 `lazy` 가 없다고 생각해보자. 우리는 지연 초기화가 필요할 때마다 각 변수에 대해 A작업을 진행하는 코드를 계속해서 작성해 주어야 한다. 

그렇다! 동일한 작업을 하는 코드를 매번 작성해야 하는 것이다. 그렇다면 우리가 할 수 있는 것은 무엇일까? 바로 `어떻게 하면 코드를 재사용할 수 있을지 고민해보는 것이다.` 실제로 우리는 위의 A작업을 `by lazy{}`를 사용함으로써 편리하게 이곳저곳에 적용해 사용하고 있다.

그렇다면 kotlin에서 제공하는 `delegated property` 는 어떻게 내부적으로 동작하는지 우선 살펴보자!

<br>

## 🔗 delegated property syntax

> `val/var <property name>: <Type> by <expression>`

delegated property를 사용하기 위해서는 위의 syntax를 따라야 한다. 이때 by 뒤에 오는 expression이 바로 <span style = "background-color:#fff5b1">위임자</span>가 된다.

그렇다면 위임자의 역할은 무엇일까? 여기서 `위임자`라는 단어에서도 알 수 있듯이 <span style = "background-color:#fff5b1">어떠한 작업을 해당 expression에게 맡기는 것이다.</span>

delegated property를 선언할 경우에는 <span style = "background-color:#fff5b1">property의 get() / set() 로직을 delegate의 getValue() / setValue() 메소드가 대신 처리해준다.</span> 따라서 delegate에서는 getValue / setValue 메소드를 제공해야 한다.

다음의 코드들을 살펴보자.

```kotlin
import kotlin.reflect.KProperty

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```

```kotlin
class Example {
    var p: String by Delegate()
}
```

우선 위임자의 역할을 할 `Delegate` 클래스가 존재한다. 위에서 말했듯이 delegated property의 위임자로 쓰이기 위해 `getValue`와 `setValue` opration을 제공하고 있다. 그리고 Example의 p property는 Delegate를 위임자로 두면서 delegated property로 선언되고 있다.

그렇다면 다음과 같이 p 변수를 읽으면 어떤 작업이 진행될까? 
```kotlin
val e = Example()
println(e.p)
```

원래는 특정 변수를 읽을 때 해당 변수의 `getter`가 호출된다. 하지만 여기서는 delegated property이기 때문에 위임자가 제공하는 `getValue`가 호출되게 된다.

```kotlin
Example@33a17727, thank you for delegating 'p' to me!
```

출력 결과를 살펴본다면 위임자에서 제공하는 getValue 함수가 호출된 것을 확인할 수 있다. 

그리고 위임자에서 제공하는 `getValue`, `setValue`는 다음과 같은 매개변수를 필수적으로 가지고 있어야 한다.

- `thisRef`: 읽는 객체 자체, 여기서는 p가 해당된다.
- `property`: p 자체에 대한 설명이 들어 있다.
- `value`: 할당될 새로운 타입 (setValue의 경우)

그럼  `e.p`에 접근해 값을 변경한다면 어떤 작업이 진행될까?

```kotlin
e.p = "NEW"
```

```kotlin
NEW has been assigned to 'p' in Example@33a17727.
```

여기서는 위임자에서 제공하는 `setValue`가 호출되는 것을 확인할 수 있다.

<br>

## 🔗 by remeber { mutableStateOf(””)}에서 위임 패턴 적용 확인!

예시 코드가 아닌 실제 우리가 사용하는 코드에서 위임 패턴의 적용을 한번 살펴보자! compose를 활용하면 다음과 같이 state를 구성하게 된다.

```kotlin
var email by remember { mutableStateOf("") }
```

다음과 같이 위임 패턴을 통해 state를 정의할 경우, `email.value` 처럼 value에 접근하지 않고도 변수명을 활용해 state를 사용할 수 있다.

그렇다면 delegate에서 제공하는 `getValue`에서 mutableState의 value를 반환해주고 있기 때문에 위와 같은 활용이 가능한 것인데,,, 한번 살펴볼까?

> `mutableStateOf` → `*createSnapshotMutableState` → `ParcelableSnapshotMutableState` → `SnapshotMutableStateImpl`

코드를 타고타고 들어가게 되면 결국 `SnapshotMutableStateImpl` 구현으로 이어지게 된다. 그리고 해당 구현 내부에 `getter`의 정의를 확인할 수 있다.

```kotlin
internal open class SnapshotMutableStateImpl<T>(
    value: T,
    override val policy: SnapshotMutationPolicy<T>
) : StateObjectImpl(), SnapshotMutableState<T> {
    @Suppress("UNCHECKED_CAST")
    override var value: T
        get() = next.readable(this).value
        set(value) = next.withCurrent {
            if (!policy.equivalent(it.value, value)) {
                next.overwritable(this, it) { this.value = value }
            }
        }
        
        private var next: StateStateRecord<T> = StateStateRecord(value).also {
        if (Snapshot.isInSnapshot) {
            it.next = StateStateRecord(value).also { next ->
                next.snapshotId = Snapshot.PreexistingSnapshotId
            }
        }
    }
        
	  //...
}
```

여기서 `T`는 `MutableState<String>` 이다. 따라서 `SnapshotMutableStateImpl`에서는 getter가 호출될 때 `mutableState의 value` 값을 반환하고 있는 것을 확인할 수 있다.

그렇다면 이제 delegated property에서 벗어나 <span style = "background-color:#fff5b1">delegate pattern</span> 자체에 대해 다뤄보도록 하자. 위임패턴의 정의는 무엇이고, 해당 패턴이 제공하는 장단점은 무엇일까? 그리고 delegated property 외 다른 활용은 어떤 것이 있을까?

<br>

## 🔗 [위임 패턴](https://en.wikipedia.org/wiki/Delegation_pattern)

우선 위임 패턴의 정의를 한번 짚고 넘어가자.

> In [software engineering](https://en.wikipedia.org/wiki/Software_engineering), the **delegation pattern** is an [object-oriented](https://en.wikipedia.org/wiki/Object-oriented_programming) [design pattern](https://en.wikipedia.org/wiki/Design_pattern) that allows [object composition](https://en.wikipedia.org/wiki/Object_composition) to achieve the same [code reuse](https://en.wikipedia.org/wiki/Code_reuse) as [inheritance](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)).
> 
> 위임 패턴은 객체 지향 디자인 패턴 중 하나이며, 상속과 같은 코드 재사용을 위한 <span style = "background-color:#fff5b1">객체 합성</span>을 가능하게 한다.
> 

여기서 주의깊게 살펴볼 개념 중 하나가 `객체 합성`이다. 위에서 언급한 것처럼 `합성`은 상속과 마찬가지로 객체지향 프로그래밍에서 널리 사용되는 코드 재사용 기법 중 하나이다. 그렇다면 `상속`과 `합성`, 두 방식의 차이는 무엇일까?
(조금 더 자세한 비교는 [다음 게시글](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5%EC%9D%98-%EC%83%81%EC%86%8D-%EB%AC%B8%EC%A0%9C%EC%A0%90%EA%B3%BC-%ED%95%A9%EC%84%B1Composition-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)을 참고하길 바란다.)

| 상속 | 합성 |
| --- | --- |
| 의존성이 `컴파일 타임`에 결정 | 의존성이 `런타임`에 결정 |
| `is-a` 관계 | `has-a` 관계 |
| 부모 클래스 구현에 `결합도가 높음` | 부모 클래스에서 제공하는 `인터페이스에 의존`(내부 구현에 의존하지 않음) |
| 클래스 사이 `정적`인 관계 | 객체 사이 `동적`인 관계 |

위의 특징을 통해서도 알 수 있듯이 상속보다는 `합성이 더욱 유연하게` 활용될 수 있다. 다시 한번 상속이 가지는 문제점을 정리해보자.

1. `클래스간 높은 결합도`
    
    부모 클래스의 기능을 하위에서 잘 활용하기 위해서는 부모 클래스의 내부 구현에 대해 상세히 알아야 한다. 이로 인해 캡슐화가 깨지고 결합도가 높아진다.
    
2. `클래스간 정적인 관계`
    
    컴파일 타임에 결정된 상속관계는 런타임에 변경될 수 없다. 따라서 유연성과 확장성이 떨어진다.
    
3. `클래스 폭발 문제`
    
    여러 기능 조합이 필요할 경우 상속을 사용할 경우, 조합의 개수가 폭발적으로 늘어나게 된다.
    

따라서 객체지향과 관련된 서적을 살펴본다면 `상속을 지양하라`라는 내용을 대부분 확인할 수 있다. <span style = "background-color:#fff5b1">추상화를 적용해야 한다면 Interface를 객체 지향 설계를 해야 한다면 합성을 상속 대신에 활용하는 것이 권장된다.</span>

그렇다면 다시 위임패턴로 돌아와서! 결국, 위임패턴은 `합성`을 통해서 상속처럼 코드를 재활용할 수 있도록 만들어주는 방식이다. 그렇다면 해당 패턴이 주는 장단점은 무엇일까? 장점은 합성이 가지는 장점과 동일하다.

### `장점`

1. 코드 재사용성 향상
    - 동일한 기능을 여러 곳에서 재사용할 수 있다.
2. 클래스 간의 결합도 낮춤
    - 각 클래스는 자신의 책임만을 집중할 수 있다.
    - ex) `by lazy{}` 를 활용한다면 개발자는 지연 초기화 작업을 신경 쓰지 않아도 된다.
3. 상속 대체
    - 다중 상속이 불가능할 경우, 위임을 통해 여러 클래스의 기능을 활용할 수 있다.

### `단점`

1. 복잡한 코드
    - 여러 객체가 상호작용하기 때문에 코드를 한번에 이해하기가 어렵다.
2. 어려운 디버깅
    - 위임된 객체를 추적해야 한다.

<br>

## 🔗 [위임 클래스 활용해보기](https://kotlinlang.org/docs/delegation.html)

위에서 다뤘던 delegated property를 통해서도 알 수 있듯이 코틀린에서는 기본적으로 `by`를 활용해 위임 패턴을 활용할 수 있다. 이를 활용한 `위임 클래스`도 한번 다루고 넘어가자!

위임 패턴을 클래스에서 활용한다면 상속 대신 활용할 수 있다. 아래 코드를 살펴보자!

```kotlin
interface BaseA {
    fun print()
}

interface BaseB {
    fun printLine()
}

class BaseAImpl(val x: Int) : BaseA {
    override fun print() {
        print(x)
    }
}

class BaseBImpl(val x: Int) : BaseB {
    override fun printLine() {
        println(x)
    }
}

class Derived(a: BaseA, b: BaseB) : BaseA by a, BaseB by b// 핵심

fun main() {
    val baseA = BaseAImpl(10)
    val baseB = BaseBImpl(20)
    val derived = Derived(baseA, baseB)

    derived.println() // 출력: 20\n
    derived.print() // 출력: 10
}
```

핵심 부분만 떼어 다시 봐보자.

```kotlin
class Derived(a: BaseA, b: BaseB) : BaseA by a, BaseB by b
```

위의 Derived 클래스는 BaseA와 BaseB interface 구현한다. 이때 `by`를 통해 그 구현을 매개변수인 `a`와 `b`에게 위임한다. 출력 결과를 확인한다면 두 클래스에서 제공하는 기능을 활용하는 것을 확인할 수 있다.

이처럼 `interface + 위임`을 활용한다면 <span style = "background-color:#fff5b1">여러 클래스에서 제공하는 기능을 마치 상속해 사용하는 것처럼 활용할 수 있게 된다.</span>

또한 상속을 사용할 때 함수를 `오버라이딩` 할 수 있는 것처럼 delegate에서 제공하는 함수를 오버라이딩해 원하는 결과를 출력할 수 있다.

```kotlin
class Derived(a: BaseA, b: BaseB) : BaseA by a, BaseB by b {
    override fun print() {
        print("오버라이딩 했지롱!")
    }
}

fun main() {
    val baseA = BaseAImpl(10)
    val baseB = BaseBImpl(20)
    val derived = Derived(baseA, baseB)

    derived.printLine() // 출력: 20\n
    derived.print() // 출력: 오버라이딩 했지롱!
}
```

<br>

### 🔗 주의할 점

```kotlin
interface Base {
    val message: String
    fun print()
}

class BaseImpl() : Base {
	override val message = "delegate 입니다!"
    override fun print() {
        print(message)
    }
}

class Derived(a: Base) : Base by a {
    override val message = "Derived 입니다!"
}

fun main() {
    val base = BaseImpl()
    val derived = Derived(base)
    
    derived.print() // 결과??
}
```

위의 코드에서 출력된 결과는 무엇일까? 과연 Derived에서 오버라이딩된 message가 출력에서 사용될까?

결과를 보면 `delegate 입니다!`가 출력되는 것을 확인할 수 있다. 당연하게도 delegate에서는 자신의 scope 내에서 구현된 message에만 접근할 수 있기 때문이다.

<br>

## 🔗 참고

- [https://kotlinlang.org/docs/delegation.html#overriding-a-member-of-an-interface-implemented-by-delegation](https://kotlinlang.org/docs/delegation.html#overriding-a-member-of-an-interface-implemented-by-delegation)
- [https://kotlinlang.org/docs/delegated-properties.html](https://kotlinlang.org/docs/delegated-properties.html)
- [https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5%EC%9D%98-%EC%83%81%EC%86%8D-%EB%AC%B8%EC%A0%9C%EC%A0%90%EA%B3%BC-%ED%95%A9%EC%84%B1Composition-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5%EC%9D%98-%EC%83%81%EC%86%8D-%EB%AC%B8%EC%A0%9C%EC%A0%90%EA%B3%BC-%ED%95%A9%EC%84%B1Composition-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)
- [https://mangkyu.tistory.com/199](https://mangkyu.tistory.com/199)
- [https://june0122.github.io/2021/08/21/design-pattern-delegate/](https://june0122.github.io/2021/08/21/design-pattern-delegate/)