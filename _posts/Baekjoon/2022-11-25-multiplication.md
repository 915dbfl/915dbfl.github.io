---
title: "[Baekjoon/🥈SilverⅠ] 1629: 곱셈⭐"
excerpt: "분할 정복으로 거듭제곱 구하기!"
categories:
  - Algorithm
tag:
  - algorithm
  - baekjoon
  - python
  - divide and conquer

last_modifeid_at: 2022-11-25
toc: true
toc_sticky: true
search: true
---

## 🙋‍♀️ [곱셈](https://www.acmicpc.net/problem/1629)

자연수 A를 B번 곱한 수를 알고 싶다. 단 구하려는 수가 매우 커질 수 있으므로 이를 C로 나눈 나머지를 구하는 프로그램을 작성하시오.

## 🙋‍♀️ 입력

첫째 줄에 A, B, C가 빈 칸을 사이에 두고 순서대로 주어진다. A, B, C는 모두 2,147,483,647 이하의 자연수이다.

## 🙋‍♀️ 출력

첫째 줄에 A를 B번 곱한 수를 C로 나눈 나머지를 출력한다.

<BR>

## 👩‍💻 Algoritm: 단순 제곱 연산

문제만 본다면 정말 간단하게 해결할 수 있는 문제이다. 직접 반복문이나 재귀를 통해 A를 B번 곱하면 되기 때문이다. 

하지만 해당 문제는 **시간 제한이 0.5초로 존재한다.** 따라서 A, B, C 모두 2,147,483,647 이하의 자연수가 될 수 있기 때문에 반복문이나 재귀를 통해 해결한다면 제한 시간 내에 문제를 해결할 수 없다.

```python
a, b, c = map(int, input().split())

print((a**b)%c)
```

<br>

## 👩‍💻 Algorithm: 반복문, 재귀를 통한 logN 풀이

**A를 B번 곱하는 풀이**는 시간복잡도가 **O(N)**이다. 

반복문와 반복문을 통해 **O(logN) 시간복잡도의 코드**를 작성할 수 있다.

우선 반복문을 통한 O(logN)의 풀이를 확인해보자!

### ⌨️ 반복문: O(logN)

```python
#반복
def power(a, b):
  ans = 1
  while b > 0:
    if b % 2 != 0:
      ans *= a
    a *= a
    b //= 2

a, b, c = map(int, input().split())
print(power(a, b))
```

### ⌨️ 재귀: O(logN)

그렇다면 이번에는 재귀를 통한 O(logN)의 풀이를 확인해보자!

```python
#재귀
def power():
  if b == 1:
    return a
  
  tmp = power(a, b//2)
  if b %2:
    return a*tmp*tmp
  else:
    return tmp*tmp

a, b, c= map(int, input().split())
print(power())
```
* 반복되는 연산값을 tmp에 미리 저장해 사용하여 연산의 수를 줄인다.
* 해당 코드는 문제를 부분 문제로 쪼갠 후, 부분 문제를 해결한 결과를 조합해 최종 결과를 얻는 `⭐분할 정복⭐` 기법이 적용되었다!

하지만 **시간복잡도가 O(logN)인 문제풀이에서도 시간초과가 발생한다.🙄** 그렇다면 어떻게 실행 속도를 더 높일 수 있을까??

<br>

## 🙋‍♀️ 파이썬에서의 곱셈: 피연산자의 크기를 줄이자!

[파이썬 피연산자의 크기가 곱셈 속도에 미치는 영향](https://www.acmicpc.net/board/view/86998)을 해당 게시글을 통해서 확인할 수 있다.

즉, 파이썬에서는 **곱셈의 피연산자의 크기가 클수록 2진수 자릿수가 늘어나게 되고 그에 따라 곱셈의 연산 속도가 느려지게 된다.**

그렇다면 이를 적용해 위 코드를 리팩토링해보자! 문제에서는 결과값이 커지는 것을 대비해 **C로 나눈 나머지를 출력하게 된다.** 따라서 우리는 **곱셈에서의 피연사자 크기를 줄이기 위해 각 단계에서 C로 나눈 나머지를 활용할 수 있다.**

```python
def power(a, b):
  if b == 1:
    return a%c
  
  tmp = power(a, b//2)
  while b > 0:
    if b%2:
      return (a*tmp*tmp)%c
    else:
      return (tmp*tmp)%c

a, b, c = map(int, input().split())
print(power(a, b))
```

<br>

# 📝 참고 자료
* <https://www.acmicpc.net/board/view/86998>
* <https://www.acmicpc.net/problem/1629>