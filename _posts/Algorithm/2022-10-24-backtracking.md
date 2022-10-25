---
title: "[Algorithm] 백트래킹"
excerpt: "DFS vs Backtracking"
categories:
  - Algorithm
tag:
  - algorithm
  - backtracking

last_modifeid_at: 2022-10-24
toc: true
toc_sticky: true
search: true
---


## 🙋‍♀️ [백트래킹(Backtracking)?](https://ko.wikipedia.org/wiki/%ED%87%B4%EA%B0%81%EA%B2%80%EC%83%89)

> **모든 조합**을 시도해서 문제의 해를 찾는다. 백트래킹은 **많은 부분 조합들을 배제**하기 떄문에 풀이 시간을 단축할 수 있다.

즉, dfs를 사용해 트리를 탐색하고, **조건에 맞지 않으면 즉시 중단**해 이전으로 돌아가는 방식을 `백트래킹(Backtracking)`이라고 한다.

<br>

## 🙌 예제) [백준: 부등호](https://www.acmicpc.net/problem/2529)

문제를 요약하면 다음과 같다.

* **>, <** 두 종류의 부등호 기호가 k개 나열된 순서열 A가 주어진다.
* **0-9** 숫자를 부등호 순서열에 만족시키게 배열한다.
  * `< < < > < < > < >`일 경우, `3 < 4 < 5 < 6 > 1 < 2 < 8 > 7 < 9 > 0`은 해당 부등호 순서열을 만족시키는 숫자들의 배열이다.
* 부등호 순서열을 만족시키는 숫자들의 최댓값, 최솟값을 출력한다.

<br>

### ✍️ 풀이

해당 문제의 경우, 모든 경우의 수를 확인하는 `브루트 포스`를 적용해야 한다.

> 순열 적용하기

```python
from itertools import permutations

k = int(input())
marks = list(input().rstrip().split())
lst = [str(n) for n in range(10)]
answer = []

for case in permutations(lst, k+1):
  for i in range(k):
    if not eval(case[i]+marks[i]+case[i+1]):
      break
  else:
    answer.append("".join(case))

answer.sort()

print(answer[-1])
print(answer[0])
```
* **permutation**을 통해서 **가능한 모든 순열**을 구한다.
* 해당 숫자들의 배열이 입력된 부등호에 만족하는지 확인한다.

하지만 이렇게 할 경우, 모든 경우를 따지기 때문에 `시간 초과`가 발생하게 된다.

<br>

> 백트래킹 적용하기

* 모든 경우의 수를 확인하지만 **조건에 만족하지 않는 경우의 수는 미리 제거한다.**

```python
k = int(input())
marks = list(input().rstrip().split())
visited = [False] * 10
max_n, min_n = "", ""

def back(lst):
  global min_n, max_n
  if len(lst) == k+1:
    if min_n == "":
      min_n = "".join(lst)
    else:
      max_n = "".join(lst)
    return

  for i in range(10):
    if len(lst) == 0:
      visited[i] = True
      back([str(i)])
      visited[i] = False
    elif visited[i] == False:
      if eval(lst[-1]+marks[len(lst)-1]+str(i)): # 만족하지 않을 경우는 포함하지 않는다. (백트래킹)
        visited[i] = True
        back(lst+[str(i)])
        visited[i] = False

back([])
print(max_n)
print(min_n)
```
* **반복문을 0-9 순으로 확인**하기 때문에 제일 처음 길이가 k+1이 되는 배열은 부등호 순서열을 만족하는 **최솟값**이 된다.
* 가장 **마지막**에 부등호 순서열을 만족하는 값이 **최댓값**이 된다.

<br>

# 📝 참고 자료
* <https://ko.wikipedia.org/wiki/%ED%87%B4%EA%B0%81%EA%B2%80%EC%83%89>
* <https://fomaios.tistory.com/entry/Algorithm-%EB%B0%B1%ED%8A%B8%EB%9E%98%ED%82%B9Backtracking%EC%9D%B4%EB%9E%80>
* <https://www.acmicpc.net/problem/2529>