---
title: "[Python] Special Method"
excerpt: "인스턴스(객체)의 비교"
categories:
  - Python
tag:
 - python
 - 스페셜 메서드
 - 매직 메서드

last_modifeid_at: 2022-07-21
toc: true
toc_sticky: true
search: true
---

🙄 객체를 만들어 사용하다보면 객체끼리의 연산, 비교 등이 필요한 경우가 있다. 자신이 원하는 방식으로 객체의 연산과 비교를 진행할 수 있도록 만들어주는 것이 바로 **스페셜 메서드 / 매직 메서드**이다!

<br>

## 👩 스페셜 메서드(Special method)?
  * **매직 메서드(Magic method)**라고도 하며 **<u>파이썬 객체들이 동일하게 가지는 인터페이스</u>**로 이해할 수 있다.
  * `객체.메서드()`의 형식으로 호출된다.
  * 앞, 뒤에 두 개의 언더바가 들어가 던더 메서드(dunder method)라고도 한다.

<br>

🙄 그럼 지금부터 몇 가지 스페셜 메서드들을 알아봅시다!

<br>

### 🧐 표준 파이썬 시퀀스 스페셜 메서드

  ```python
  class Fruit:
    def __init__(self, name, types): # 1. __init__()
      self.name = name
      self.types = types

    def __len__(self): # 2. __len__()
      return len(self.types)

    def __getitem__(self, position): # 3. __getitem__()
      return self.types[position]
  ```

-----

<br>

### 🧐 수치형 스페셜 메서드

  ```python
    # 덧셈
    def __add__(self, other):
      return len(self.types) + len(other.types)

    # 뺄셈
    def __sub__(self, other):
      ...

    # 곱셈
    def __mul__(self, other):
      ...
    
    # 나눗셈
    def __mul__(self, other):
      ...
  ```

-------

<br>

### 🧐 비교를 위한 스페셜 메서드

  ```python
  # <
  def __lt__(self, other):
    return len(self.types) < len(other.types)

  # <=
  def __le__(self, other):
    ...

  # >
  def __gt__(self, other):
    ...

  # >=
  def __ge__(self, other):
    ...

  # ==
  def __eq__(self, other):

  # !=
  def __ne__(self, other):
  ```

  * 위의 메소드들을 구현한다면 **객체 사이의 비교**가 가능해집니다. 따라서 **객체들로 이루어진 리스트**의 **정렬** 또한 가능해집니다!

  <br>

## 📃 참고
* <https://wikidocs.net/89>
* <https://dev-dain.tistory.com/98>
* <https://medium.com/humanscape-tech/%ED%8C%8C%EC%9D%B4%EC%8D%AC%EC%9D%98-%EC%8A%A4%ED%8E%98%EC%85%9C-%EB%A9%94%EC%84%9C%EB%93%9C-special-method-2aea6bc4f2b9>
