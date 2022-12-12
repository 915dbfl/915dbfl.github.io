---
title: "[Baekjoon/🥇Ⅴ] 9251: 숨바꼭질3"
excerpt: "0-1 bfs or dijkstra"
categories:
  - Algorithm
tag:
  - algorithm
  - baekjoon
  - python
  - 0-1 bfs
  - dijkstra

last_modifeid_at: 2022-12-12
toc: true
toc_sticky: true
search: true
---

## 🙋‍♀️ [숨바꼭질3](https://www.acmicpc.net/problem/13549)

수빈이는 동생과 숨바꼭질을 하고 있다. 수빈이는 현재 점 N(0 ≤ N ≤ 100,000)에 있고, 동생은 점 K(0 ≤ K ≤ 100,000)에 있다. 수빈이는 걷거나 순간이동을 할 수 있다. 만약, 수빈이의 위치가 X일 때 걷는다면 1초 후에 X-1 또는 X+1로 이동하게 된다. 순간이동을 하는 경우에는 0초 후에 2*X의 위치로 이동하게 된다.

수빈이와 동생의 위치가 주어졌을 때, 수빈이가 동생을 찾을 수 있는 가장 빠른 시간이 몇 초 후인지 구하는 프로그램을 작성하시오.

## 🙋‍♀️ 입력

첫 번째 줄에 수빈이가 있는 위치 N과 동생이 있는 위치 K가 주어진다. N과 K는 정수이다.

## 🙋‍♀️ 출력

수빈이가 동생을 찾는 가장 빠른 시간을 출력한다.

<BR>

## 👩‍💻 Algoritm: bfs 적용

수빈이가 동생을 찾는 **가장 빠른 시간**을 구하는 문제이다. 따라서 **최단 거리**를 찾는 문제를 해결하기 위해 **bfs**를 적용하였다.

### ⌨️ bfs

```python
from collections import deque
import sys

def bfs():
  dq = deque([[s, 0]])
  visited = set([s])

  while dq:
    cur, time = dq.popleft()

    if cur == b:
      print(time)
      return
    else:
      if cur < b:
        if cur*2 not in visited:
          dq.append([cur*2, time])
          visited.add(cur*2)
        if cur+1 not in visited:
          dq.append([cur+1, time+1])
          visited.add(cur+1)
      if cur > 0 and cur-1 not in visited:
        dq.append([cur-1, time+1])
        visited.add(cur-1)

s, b = map(int, sys.stdin.readline().split())
if b < s:
  print(s-b)
else:
  bfs()
```

하지만 이렇게 짜고 결과를 확인해보니 **오답**이었다.
이유가 무엇일까 고민을 하던 중 [한 게시글](https://www.acmicpc.net/board/view/38887#comment-69010)을 통해서 그 원인을 파악할 수 있었다.

우선 그래프 탐색 문제에서 사용할 수 있는 알고리즘을 다시 한 번 정리해보자!

> ⭐ 그래프 탐색 
* **모든 가중치 동일**
  * dfs/bfs
* **한 노드**에서 다른 노드까지의 최단 거리
  * **음의 가중치** 존재
    * 벤만-포드 알고리즘
  * 양의 가중치만 존재
    * 다익스트라 알고리즘
* **모든 노드 사이**의 최단 거리
  * 플로이드 워샬 알고리즘

그렇다! 원래 **bfs/dfs**는 **모든 간선의 가중치가 동일**하다는 전제조건이 있어야 한다. 따라서 가중치가 **0/1**인 이 문제는 단순 bfs를 통해 해결할 수 없다.

이를 해결할 수 있는 방법은 **⭐가중치가 0인 경우를 큐의 맨 앞에 넣어줌으로써⭐** 우선으로 처리하는 것이다. 그렇다면 아래 코드를 통해 자세히 살펴보자!!

### ⌨️ 0-1 bfs⭐

```python
from collections import deque
import sys

def bfs():
  dq = deque([[s, 0]])
  visited = set([s])

  while dq:
    cur, time = dq.popleft()

    if cur == b:
      print(time)
      return
    else:
      if cur < b:
        if cur*2 not in visited:
          dq.appendleft([cur*2, time]) # ⭐⭐⭐
          visited.add(cur*2)
        if cur+1 not in visited:
          dq.append([cur+1, time+1])
          visited.add(cur+1)
      if cur > 0 and cur-1 not in visited:
        dq.append([cur-1, time+1])
        visited.add(cur-1)

s, b = map(int, sys.stdin.readline().split())
if b < s:
  print(s-b)
else:
  bfs()
```

<br>

## 👩‍💻 Algoritm: 다익스트라

해당 문제는 모든 간선의 가중치가 동일하지 않기 때문에 **다익스트라**를 활용해 문제를 풀이하는 것이 좋다!

[다익스트라](https://915dbfl.github.io/algorithm/kevin_bacon/)는 위에서  간단히 설명했듯이 **하나의 정점에서 다른 모든 정점까지의 최단 거리를 구하는** 알고리즘이다.

그렇다면 이를 활용한 풀이를 확인해보자!!

```python
import sys
from heapq import heappop, heappush

inf = sys.maxsize # 정수 최댓값

s, b = map(int, sys.stdin.readline().split())
dp = [inf]*100001
heap = []

def dijkstra(s, b):
  if b <= s:
    print(s-b)
    return

  heappush(heap, [0, s])
  while heap:
    t, c = heappop(heap)
    if c == b:
      print(t)
      return
    else:
      for nx in [c*2, c+1, c-1]:
        if 0<= nx <= 100000:
          if nx == c*2 and dp[nx] == inf:
            dp[nx] = t
            heappush(heap, [t, nx])
          elif dp[nx] == inf:
            dp[nx] = t+1
            heappush(heap, [t+1, nx])

dijkstra(s, b)
```

<br>

## 📝 참고 자료
* <https://www.acmicpc.net/problem/13549>
* <https://www.acmicpc.net/board/view/38887#comment-69010>
* <https://velog.io/@hamfan524/%EB%B0%B1%EC%A4%80-13549%EB%B2%88-Python-%ED%8C%8C%EC%9D%B4%EC%8D%AC-Dijkstra>