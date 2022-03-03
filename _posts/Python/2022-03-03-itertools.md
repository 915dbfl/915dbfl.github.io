---
title: "[Python] 파이썬 순열과 조합"
excerpt: "itertools를 사용한 permutation, combination, product"
categories:
  - Python
tag:
 - python
 - permutation
 - combination
 - product

last_modifeid_at: 2022-03-03
toc: true
toc_sticky: true
search: true
---

## 파이썬 순열과 조합👩
  * 파이썬에서 주어진 리스트에 대해서 순열과 조합을 구할 경우, **itertools**를 사용하여 손쉽게 그 값들을 구할 수 있다.

<br>

## 🙋‍♀️ permutations: 순열(nPr)
   * 순열이란 서로 다른 n개 중에서 r개를 취해 순서를 정해 나열한 것이다.
   * 즉, **순서**가 중요하여 (a, b)와 (b, a)는 서로 **다른** 것으로 간주한다.
   * permutations(반복 가능한 객체, r)

   ```
   from itertools import permutations

    a = [1, 2, 3]
    b = permutations(a)
    # [(1, 2, 3), (1, 3, 2), (2, 1, 3), (2, 3, 1), (3, 1, 2), (3, 2, 1)]

   ```

<br>

## 🙋‍♀️ combinations: 조합(nCr)
  * 조합은 서로 다른 n개 중에서 r개를 취해 순서를 고려하지 않고 조를 만드는 것이다.
  * 즉, 순서는 중요하지 않으므로 (a, b)와 (b, a)는 서로 **동일한** 것으로 간주한다.
  * combinations(반복 가능한 객체, r)

  ```
  from itertools import combinations

  a = [1, 2, 3]
  b = combinations(a, 2)
  # [(1, 2), (1, 3), (2, 3)]

  ```

<br>

## 🙋‍♀️ product: 데카르트 곱
 * **두 개 이상의 리스트**의 **모든 조합**을 구할 수 있다.

### 하나의 리스트 속 여러 문자열에 대한 데카르트 곱 👩
  ```
  from itertools import product

    a = ["abc", "!@#"]
    b = product(*a)
    # ('a', '!'), ('a', '@'), ('a', '#'), ('b', '!'), ('b', '@'), ('b', '#'), ('c', '!'), ('c', '@'), ('c', '#')

  ```
    * 리스트 a에 있는 두 문자열에 대한 곱집합을 구하는 과정이다.

<br>

### 여러 리스트에 대한 데카르트 곱 👩
  ```
  from itertools import product

  a = ["abc", "!@#"]
  b = [1, 2]
  ab = product(a, b)

  #('abc', 1), ('abc', 2), ('!@#', 1), ('!@#', 2)

  ```
  * 두 리스트의 모든 조합을 구하는 과정이다.

<br>

## 🙋‍♀️ product(반복 가능한 객체, reapeat = ?) : 중복 순열
  * product에 **repeat**을 설정하면 중복 순열을 구할 수 있게 된다.
  
  ```
  from itertools import product

  a = ["abc", "!@#"]
  b = product(a, repeat = 2)
  # [('abc', 'abc'), ('abc', '!@#'), ('!@#', 'abc'), ('!@#', '!@#')]

  ```
  * 즉, **["abc", "abc", "!@#", "!@#"]**을 가지고 순열을 구한 것과 동일한 결과를 확인 할 수 있다.

  

## 📃 참고
* [https://prod.velog.io/@hyun0820/Python-%EC%88%9C%EC%97%B4-%EC%A1%B0%ED%95%A9-%EA%B3%B1%EC%A7%91%ED%95%A9-%EC%A4%91%EB%B3%B5-%EC%88%9C%EC%97%B4-%EC%A4%91%EB%B3%B5-%EC%A1%B0%ED%95%A9-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC](https://prod.velog.io/@hyun0820/Python-%EC%88%9C%EC%97%B4-%EC%A1%B0%ED%95%A9-%EA%B3%B1%EC%A7%91%ED%95%A9-%EC%A4%91%EB%B3%B5-%EC%88%9C%EC%97%B4-%EC%A4%91%EB%B3%B5-%EC%A1%B0%ED%95%A9-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC)
* [https://velog.io/@davkim1030/Python-%EC%88%9C%EC%97%B4-%EC%A1%B0%ED%95%A9-product-itertools](https://velog.io/@davkim1030/Python-%EC%88%9C%EC%97%B4-%EC%A1%B0%ED%95%A9-product-itertools)
* [https://velog.io/@dramatic/Python-permutation-combination-%EC%88%9C%EC%97%B4%EA%B3%BC-%EC%A1%B0%ED%95%A9](https://velog.io/@dramatic/Python-permutation-combination-%EC%88%9C%EC%97%B4%EA%B3%BC-%EC%A1%B0%ED%95%A9)