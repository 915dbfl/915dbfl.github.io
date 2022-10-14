---
title: "[Algorithm] 그래프 탐색"
excerpt: "플로이드, 다익스트라"
categories:
  - Algorithm
tag:
  - algorithm
  - floyd
  - Dijkstra

last_modifeid_at: 2022-10-14
toc: true
toc_sticky: true
search: true
---

이번 시간에는 그래프 탐색 중 두가지 방법인 `플로이드`와 `다익스트라`에 대해 자세히 다뤄보고자 한다!


# 🙋‍♀️ 플로이드-워셜 vs 다익스트라

* 플로이드 워셜(Floyd-Warshall)
  * **모든 정점**에서 **다른 모든 정점**까지의 최단 경로를 구하는 알고리즘
* 다익스트라(Dijkstra)
  * **하나의 정점**에서 **다른 모든 정점**까지의 최단 거리를 구하는 알고리즘

<br>

## 🙌 Floyd-Warshall
* [과정](https://www.acmicpc.net/problem/1389)
  * **2차원 인접 행렬**을 구성한다.
    * **모든 정점에서 다른 모든 정점까지의 최단 거리**를 구하기 때문에 2차원 인접 행렬을 필요로 한다.
    * 연결된 정점끼리의 **가중치**를 입력한다.
      * **자기 자신**에 대한 가중치는 **0**, **연결되지 않은 정점**끼리의 가중치는 **INF**로 설정한다.
  * 각 라운드마다 **중간 노드**로 사용할 수 있는 노드를 선택한다.
    * 중간 노드를 거치는 방법과 안거치는 방법 중 더 나은 방법을 선택한다.

다음의 사진을 보면 위 과정을 조금 더 자세히 이해할 수 있을 것이다!!

<img src = "https://drive.google.com/uc?id=18iipXi3GNVvVKi4Hizuwu2Qr_YJsZQpp" width = 600 height = 500>

그렇다면 이제 위의 과정을 토대로 플로이드 워셜 알고리즘을 작성해보려고 한다!! 그 예로는 백준의 [케빈 베이컨의 6단계 법칙](https://www.acmicpc.net/problem/1389)을 사용하겠다!!

<br>

### ⌨️ 플로이드-워셜 알고리즘

```python
import sys

N, M = map(int, sys.stdin.readline().split())
graph = [[N for _ in range(N)] for _ in range(N)]

for i in range(M):
  a, b = map(int, sys.stdin.readline().split())
  graph[a-1][b-1] = 1
  graph[b-1][a-1] = 1
    
for i in range(N):
  for j in range(N):
    for k in range(N):
      if j == k:
        graph[j][k] = 0
      else:
        graph[j][k] = min(graph[j][k], graph[j][i] + graph[i][k])
                
answer = []
for row in graph:
  answer.append(sum(row))
print(answer.index(min(answer))+1)
```

여기서 코드를 나눠 분석해보자!!!

```python
for i in range(M):
  a, b = map(int, sys.stdin.readline().split())
  graph[a-1][b-1] = 1
  graph[b-1][a-1] = 1
```
* 해당 부분은 **2차원 인접 행렬**을 나타내는 부분이다.
* 문제에서는 모든 가중치가 **1**로 표현되므로 연결되는 정점 사이의 가중치를 1로 입력한다.

```python
for i in range(N): # 중간노드가 될 i
  for j in range(N):
    for k in range(N):
      if j == k:
        graph[j][k] = 0
      else:
        graph[j][k] = min(graph[j][k], graph[j][i] + graph[i][k]) # i를 거치는 경우와 안거치는 경우를 비교한다.
```
* **삼중 포문**을 사용하기 때문에 시간 복잡도가 `O(N^3)`이다.
  * 따라서 그래프의 크기가 **⭐작은 경우⭐**만 해당 알고리즘을 사용해야 한다.

<br>

## 🙌 Dijkstra
* [과정](https://velog.io/@tks7205/%EB%8B%A4%EC%9D%B5%EC%8A%A4%ED%8A%B8%EB%9D%BC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-with-python)
  * 하나의 정점을 시작으로 이와 연결된 정점 중 가중치가 가장 작은 노드를 선택한다.
  * 다른 노드들의 값을 갱신한다.
  * 모든 노드가 선택될 때까지 위의 두 과정을 반복한다.

이 과정 또한 다음의 그림을 통해 이해하면 쉬울 것이다!!

<img src = "https://drive.google.com/uc?id=1txtM487W34XbCymmWhV5J4Dy-jfRzG8K" width = 600 height = 500>

그렇다면 위의 과정을 토대로 다익스트라 알고리즘을 작성해보자!!!

<br>

### ⌨️ 다익스트라 알고리즘

다익스트라 알고리즘에서는 매 단계마다 **가장 작은 가중치**를 선택하는 작업이 필요하다!! 따라서 우리는 `heapq(최소힙)`을 적용해 가장 작은 값을 빠르게 찾고자 한다!!

```python
from heapq import heappop, heappush

def dijkstra(start):
  q = []
  heappush(q, (0, start)) # (우선순위(거리), 값) 형태로 push
  distance[start] = 0

  while q:
    dis, cur = heappop(q)

    if distance[cur] < dis: # 입력된 값이 노드의 가중치보다 작을 경우 이미 방문했으므로 pass
      continue
    
    for i in graph[cur]:
      if dis + i[1] < distance[i[0]]:
        distance[i[0]] = dis + i[1]
        heappush(q, (dis+i[1], i[0]))

n, m, k = map(int, input().split()) # n: 노드의 개수/ m: 간선의 개수/ k: 시작 노드
INF = 1e8

graph = [[] for _ in range(n+1)]
distance = [n] * (n+1)

for _ in range(m):
  u, v, w = map(int, intput().split()) # u: 출발 노드/ v: 도착 노드/ w: 가충치
  graph[u].append((v, w))

dijkstra(k)
print(distance)
```
<br>


# 📝 참고 자료
* <https://chanhuiseok.github.io/posts/algo-50/>
* <https://reakwon.tistory.com/55>
* <https://velog.io/@tks7205/%EB%8B%A4%EC%9D%B5%EC%8A%A4%ED%8A%B8%EB%9D%BC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-with-python>
* <https://www.acmicpc.net/problem/1389>