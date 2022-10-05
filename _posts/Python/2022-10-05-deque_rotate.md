---
title: "[Python] deque: rotate 함수"
excerpt: "원형 회전을 쉽게 구현하는 법!"
categories:
  - Python
tag:
 - python
 - deque
 - deque.rotate

last_modifeid_at: 2022-10-05
toc: true
toc_sticky: true
search: true
---

🙂 deque의 흥미로운 함수 `rotate()`에 대해 짚고 넘어가고자 한다.

## 👩 [deque.rotate()](https://docs.python.org/ko/3/library/collections.html#collections.deque)?

* 단어 그대로 deque에 들어있는 요소들을 **회전**할 수 있는 함수이다.
  * rotate(**n<0**): n단계 **오른쪽**으로 회전
  * roate(**n>0**): n단계 **왼쪽**으로 회전

  <br>

그 사용법을 한 예제를 통해 자세히 이해해보자!!


<br>

## 📝백준 11866: 요세푸스 문제0
요세푸스 문제는 다음과 같다.

1번부터 N번까지 N명의 사람이 원을 이루면서 앉아있고, 양의 정수 K(≤ N)가 주어진다. 이제 순서대로 K번째 사람을 제거한다. 한 사람이 제거되면 남은 사람들로 이루어진 원을 따라 이 과정을 계속해 나간다. 이 과정은 N명의 사람이 모두 제거될 때까지 계속된다. 원에서 사람들이 제거되는 순서를 (N, K)-요세푸스 순열이라고 한다. 예를 들어 (7, 3)-요세푸스 순열은 <3, 6, 2, 7, 5, 1, 4>이다.

N과 K가 주어지면 (N, K)-요세푸스 순열을 구하는 프로그램을 작성하시오.

<br>

### 👩 입력
첫째 줄에 N과 K가 빈 칸을 사이에 두고 순서대로 주어진다. (1 ≤ K ≤ N ≤ 1,000)

<br>

### 👩 출력
예제와 같이 요세푸스 순열을 출력한다.

<br>

### 👩 입출력 예
* 입력

> 7 3

* 출력

> <3, 6, 2, 7, 5, 1, 4>

<br>

### 🙋‍♀️ 풀이: deque.rotate() 사용!

```python
from collections import deque

n, k = map(int, input().split())
dq = deque(range(1, n+1))

print("<", end = "")

for i in range(n):
  dq.rotate(-k)
  if i == n-1:
    print(dq.pop(), end = ">")
  else:
    print(dq.pop(), end = ", ")
```

* 예시에서 n이 7이므로 생성되는 deque는 `deque([1, 2, 3, 4, 5, 6, 7])`이다.
* k = 3이므로 `rotate(-3)`을 적용하면 다음과 같다!
  * `deque([4, 5, 6, 7, 1, 2, 3])`

<br>

## 📃 참고
* <https://www.acmicpc.net/problem/11866>
* <https://docs.python.org/ko/3/library/collections.html#collections.deque>

