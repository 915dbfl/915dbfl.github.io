---
title: "[Clean Architecture] domain/business logic 분리?"
excerpt: "domain/business logic이 뭘까? 왜 분리해야 할까?"
categories:
  - logic
tag:
 - domain logic
 - business logic


last_modifeid_at: 2023-04-06
toc: true
toc_sticky: true
search: true
---

<br>

안드로이드 패키지 분리에 대해 알아보다 자주 등장하는 단어가 보였다. 바로 `domain`!

그렇다면 domain이란 무엇일까??🤔

또한, 코딩을 하다보면 **코드간의 의존성을 줄이기 위해 비지니스 로직을 분리하라**는 말을 많이 들어봤을 것이다!

가끔 내가 작성하는 부분이 비지니스 로직인지 아닌지가 헷갈릴 때가 많다! 지금부터 domain, business 그 의미를 명확히 파악해보고자 한다.

<br>

# 🙋🏻‍♀️ domain, business?

소프트웨어 공학에서의 domain과 business는 어떤 의미를 가질까??


## ✓ domain 로직 = business 로직

쉽게 설명하고자 stackoverflow의 말 몇 개를 가져와봤다.

> Domain is what you are modeling

> if you are working on say a flight reservation system, the application domain would be flight reservations.

위 두 문장을 통해 `domain`이란 **모델링한 소프트웨어의 존재 목적**이라 생각할 수 있다.
* 만약 비행기표 예약 서비스가 있다면 그 application의 도메인이 비행기표 예약인 것처럼 말이다.

<br>

[아래 그림](https://enterprisecraftsmanship.com/posts/what-is-domain-logic/)을 통해 조금 더 명확히 이해해보자!

<img src = "https://drive.google.com/uc?id=1FlfVBaVraxtS7GkeVL5GJ51qUEh1-dGR" width = 600 height = 500>

* **우리가 해결하고자 하는 문제 = 우리가 개발하는 소프트웨어의 존재 목적 = domain**인 것을 확인할 수 있다.

* 그 옆에 존재하는 solution space에는 문제를 해결하기 위한 다양한 요소들이 포함되며, 해당 요소들은 동의어로 바꾸어 사용할 수 있다.
  * 결국 **domain을 해결하는 코드들이 business, domain 로직에 해당하고, 이 둘은 동의어로 생각할 수 있다.**

<br>

# 🙋🏻‍♀️ domain logic vs application service logic

위에서도 말했듯이 코드를 짜다보면 business logic을 분리해 코드간의 의존성을 낮추는 것은 매우 중요하다.

그렇다면 정확히 어떤 부분을 우리는 business logic으로 분리해야 할까?


> whether or not the code **makes decisions that have a business meaning.**
* ⭐️domain에 대한 의사결정을 하고 있는가?⭐️가 이를 구분하는 핵심이 되게 된다.

<br>

# 🙋🏻‍♀️ 인출 기능 예시로 domain과 application service logic을 분리해보자!

인출과 관련해서 다음의 코드들이 존재한다고 생각해보자!

1. 계좌 잔액이 충분한지 확인한다.
2. 유효하다면 인출 버튼이 활성화되고, 유효하지 않는다면 에러 메세지를 띄운다.
3. 사용자의 잔액을 감소시킨다.
4. 사용자의 잔액을 db에 저장한다.

여기서 domain logic과 application service logic을 분리해보자!!

핵심은 **domain 즉, 문제에 대한 의사결정을 하고 있는가**인 점을 생각해라!

## ✓ domain logic
* 계좌 잔액이 충분한지 확인 -> 인출이 가능한지에 대한 의사결정
* 사용자의 잔액 감소 -> 인출 서비스 수행

## ✓ application service logic
* 유효하지 않으면 에러 메세지 출력 -> ui
* 잔액을 db에 저장한다. -> persistence


## ✓ 애매하다?
만약 위의 1, 2번 과정이 합쳐져 '계좌 잔액이 부족하면 에러 메세지를 띄운다.'의 코드가 존재한다고 생각해보자!

이를 domain으로 분리할지, application service로 분리할지가 애매해진다.

그렇기 때문에 애매하다면 함수를 분리하고 코드를 쪼개보자!!

<br>

# 🙋🏻‍♀️ 로직의 분리 -> 관심사 분리

위에서 말한 것처럼 domain logic을 분리하면 개발자가 db나 ui와 같은 기술적 구현 사항에 신경쓰지 않고, domain logic을 이해할 수 있게 되며, 이는 변경사항이 있을 경우, 기능 수정과 추가 용이로 이어진다.

(클린 아키텍처를 위해서도 domain logic 분리는 매우 중요하다. 필자의 경우, 클린 아키텍처에 대해 자세히 학습하지 못해 추후 학습을 진행할 예정이다!)

<br>

# 📃 참고
* <https://velog.io/@eddy_song/domain-logic>
* <https://enterprisecraftsmanship.com/posts/what-is-domain-logic/>
* <https://stackoverflow.com/questions/360860/what-is-domain-logic>