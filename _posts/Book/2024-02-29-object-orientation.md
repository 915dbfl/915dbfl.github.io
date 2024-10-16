---
title: "[개발 서적] 객체지향의 사실과 오해(들어가며)"
excerpt: "part1. 객체지향에 대해 쉽지만 명확하게 이해하기"
categories:
  - Book
tag:
  - 개발 서적
  - 객체지향의 사실과 오해
  - 1장

last_modified_at: 2024-02-29
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며
`객체지향`, `solid 원칙`, `캡슐화` 등을 아무렇지 않게 사용하고 있지만 정작 '객체지향이 그래서 정확히 뭐야?'라고 스스로 물어봤을 때 명확한 정의를 내릴 수 없었다. 우연히 아는 지인이 '객체지향의 사실과 오해', '오브젝트'라는 개발 서적을 추천해줘 취준 기간 동안 책을 통해 기본적인 개념들을 명확히 이해하고 정리해보고자 한다.

어떻게 글을 정리할까 고민을 하다 장별로 정리하지 않고 내 나름대로의 소주제와 그 안의 내용들로 정리해보기로 했다.

<br>

# 🔗 객체지향 = 실세계 모방

객체지향을 "실세계를 직접적이고 직관적으로 모델링할 수 있는 패러다임"이라는 설명은 참 와닿지가 않는다.

하지만 객체지향을 가장 쉽게 이해할 수 있는 방식이 `객체지향 = 실세계 모방`이기도 하다.

책에 나온 예시를 토대로 객체 지향에서의 중요 개념을 모두 다룰 수 있다.

| 책에 나온 예시처럼 `출근길 커피를 주문해 받는 과정`을 생각해보자
고객, 캐셔, 바리스타는 각각 다음의 역할을 수행한다.
  - 고객
    - 커피 주문을 한다.
    - 제조가 완료된 커피를 받는다.
  - 캐셔
    - 고객 주문을 받는다.
    - 주문내역을 바리스타에게 전달한다.
    - 완성된 음료를 고객에게 전달한다.
  - 바리스타
    - 주문 내역을 받는다.
    - 음료를 제조한다.
    - 제조한 음료를 캐셔에게 전달한다.

<br>

# 🔗 객체지향에서의 협력, 역할, 책임

우리는 위의 예시에서 객체지향에서 중요한 세 가지 개념을 모두 이해할 수 있다.

## [객체들은 `협력`한다]
`협력`,`요청(request)`, `응답(response)`

위 예시에서 고객, 캐셔, 바리스타는 각각 하나 객체에 해당한다. 이들은 <span style = "background-color:#fff5b1">협력</span>을 통해 커피 주문이라는 목표를 수행한다.

또한, 이들은 서로 `요청`, 요청에 대한 `응답`을 하며 협력을 진행한다.


## [객체는 `역할`과 `책임`이 있다]

각각의 객체들은 역할과 책임이 존재한다. 책임은 역할에 따라 따라오는 것이라 생각하면 된다. 선생님이라는 역할은 학생들을 가르치는 책임이 있는 것처럼 말이다.

그리고 우리는 여기서 역할과 책임의 특징도 짚고 넘어갈 수 있다.
- <span style = "background-color:#fff5b1">여러 객체가 동일한 역할을 수행할 수 있다.</span>
  - 여러 캐셔가 로테이션으로 주문을 받는다 해도 커피 주문 과정에서는 아무런 영향을 미치지 않는다.
- <span style = "background-color:#fff5b1">즉, 역할은 대체 가능하다.</span>
- <span style = "background-color:#fff5b1">책임을 수행하는 방법은 자율적이다. </span>
  - 바리스타는 원하는 방식대로 커피를 제조할 수 있다.
- <span style = "background-color:#fff5b1">한 사람이 동시에 여러 역할을 수행할 수 있다.</span>
  - 캐셔가 바리스타의 역할까지 수행할수도 있는 것이다.

<br>

# 🔗 객체의 특징은 무엇일까?

그렇다면 객체는 역할과 책임이 있고, 협력을 한다. 이렇게 되면 객체는 다음의 두가지 특징을 가지게 된다.

1. 협력적
  - 다른 객체와 조화롭게 협력할 수 있도록 개방적이어야 한다.
2. 자율적
  - 협력에 참여하는 방법을 스스로 결정할 수 있어야 한다.

## [`상태`와 `행동`을 지닌 자율적인 객체?]

객체의 자율성에 대해 조금 더 자세히 다뤄보자.

> 객체의 자율성은 객체의 내부와 외부를 명확하게 구분하는 것으로부터 나온다.

- 객체는 역할에 따른 책임을 다하기 위해 <span style = "background-color:#fff5b1">상태와 행동을 스스로 지닌다.</span>
- 이러한 상태와 행동을 외부에서 통제할 수 있다면 객체는 자율성을 뺏긴다.
- 따라서, 객체는 사적인 부분은 외부와 <span style = "background-color:#fff5b1">분리해 스스로 관리하면서 자율성을 갖게 된다.</span>
  - 이는 추후, 캡슐화 개념과 관련되게 된다.

<br>

# 🔗 객체 지향 != 클래스, 상속

`객체지향`하면 가장 먼저 떠오르는 것이 클래스와 상속일 것이다. 물론 나도 그랬다. 하지만 <span style = "background-color:#fff5b1">[객체]지향</span>인 것처럼 객체지향의 중심에는 객체가 있다!

우리는 다음을 알아야 한다.
1. <span style = "background-color:#fff5b1">클래스는 객체의 구현 매커니즘에 불과하다.</span>
2. 객체지향에서는 <span style = "background-color:#fff5b1">클래스의 정적인 관계가 아니라 메세지를 주고받는 객체들의 동적인 관계</span>가 중심이다.

<br>

# 🔗 객체지향의 본질
이 내용은 책에서 정리한 내용을 가져온 것이다.

- 객체 지향이란 시스템을 <span style = "background-color:#fff5b1">상호작용하는 자율적인 객체들의 공동체</span>로 바라보고 객체를 이용해 시스템을 분할하는 방법이다.
- 자율적인 객체란 <span style = "background-color:#fff5b1">상태</span>와 <span style = "background-color:#fff5b1">행위</span>를 함께 지니며 스스로 자기 자신을 책임지는 객체를 의미한다.
- 객체는 시스템의 행위를 구현하기 위해 다른 객체와 <span style = "background-color:#fff5b1">협력</span>한다. 각 객체는 협력 내에서 정해진 <span style = "background-color:#fff5b1">역할</span>을 수행하며 역할은 관련된 <span style = "background-color:#fff5b1">책임</span>의 집합이다.
- 객체는 다른 객체와 협력하기 위해 메시지를 전송하고, <span style = "background-color:#fff5b1">메시지</span>를 수신한 객체는 메시지를 처리하는 데 적합한 <span style = "background-color:#fff5b1">메서드</span>를 자율적으로 선택한다.