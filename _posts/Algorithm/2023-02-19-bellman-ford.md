---
title: "[Algorithm] Bellman-Ford"
excerpt: "음의 간선이 존재할 때 최단거리 찾기"
categories:
  - Algorithm
tag:
  - algorithm
  - bellman ford

last_modifeid_at: 2023-02-19
toc: true
toc_sticky: true
search: true
---


## 🙌 벨만포드(Bellman-Ford)?
벨만포드의 경우에도 다익스트라와 마찬가지로 **시작 정점으로부터 다른 정점까지의 최단 경로**를 구하는 알고리즘이다.

<br>

## 🙌 벨만포드 특징
* **음의 가중치**가 있는 그래프에서의 최단 거리를 구할 수 있다.
* 음의 가중치로 인한 **음의 사이클 존재 여부**를 알 수 있다.

<br>

## 🙌 음의 사이클 여부는 어떻게 알 수 있을까?
그렇다면 음의 사이클 존재 여부를 어떻게 확인하는 것일까?
* 전체 코드의 반복을 **V-1**만 진행한다.
* **정점의 개수가 V일 때** 시작 정점에서 특정 정점까지 도달하기 위해 거칠 수 있는 **최대 간선의 개수는 V-1**이기 때문이다.
* 따라서 V번째 간선이 존재한다면 이는 음의 사이클로 판단된다.

<br>

## 🙌 진행 과정
코드를 보기 전에 진행 과정을 한 번 짚고 넘어가자!!
1. 시작 정점을 제외하고 다른 정점까지의 거리를 모두 무한으로 초기화한다.
2. 정점 수(v-1) 만큼 다음의 과정을 반복한다.
  * 한 번의 반복마다 모든 간선을 확인한다.
  * 현재 정점에서 인접한 모든 정점을 탐색한다.
  * 기존 저장되어 있는 인접 정점까지의 거리보다 현재 정점을 거쳐 이동하는 거리가 더 짧을 경우 갱신한다.
3. 마지막 v-1 반복 후에도 거리가 갱신된다면 음수 순환이 존재하는 것으로 판단한다.

<br>

그렇다면 이제 코드를 통해 다시 한 번 그 과정을 확인해보자!!

```python
import sys
input = sys.stdin.readline
INF = int(1e9) 

v, e = map(int, input().split())
edges = []
dist = [INF] * (v+1) # 무한대로 거리 초기화

for _ in range(e):
    u, v, w = map(int, input().split())
    edges.append((u, v, w))

def bellmanFord(start):
    dist[start] = 0 # 시작 정점 거리 0으로 초기화

    # 최대 간선의 개수 + 1인 v 만큼 반복을 수행
    # 마지막 v번 째에 변경된다면 negative cycle!
    for i in range(v):
        # 매 반복마다 모든 간선 확인
        for j in range(e):
            node = edges[j][0]
            next_node = edges[j][1]
            cost = edges[j][2]
            
            # 현재 간선을 거쳐 이동하는 거리가 더 짧을 경우
            if dist[node] != INF and dist[next_node] > dist[node] + cost:
                dist[next_node] = dist[node] + cost

                # v번째에 갱신되는 값이 있을 경우 negative cycle!
                if i == v-1:
                    return True
    return False

if bellmanFord(1):
    for i in range(2, v+1):
        if dist[i] == INF:
            print("도달할 수 없다.")
        else:
            print(dist[i])
else:
    print("Negative Cycle Exist")
```

<br>

## 👀 단순 음의 사이클만 확인하는 경우라면?
시작 정점으로 부터 최단 거리를 확인하는 것이 아닌 **그래프 상에 음의 사이클 존재 여부만을 확인**하면 되는 경우!

굳이 시작 정점을 둘 필요가 없어진다!!

따라서 **시작 정점과 인접한 정점인지 판단하는 부분을 없애면 되는 것이다!**

```python
    if dist[next_node] > dist[node] + cost:
```

<br>

## 🙌 시간복잡도
* 각 반복마다 모든 간선을 확인한다. (E)
* 정점의 개수 만큼 반복을 진행한다. (V)

-> O(VE)

<br>

# 📝 참고 자료
* <https://velog.io/@younge/Python-최단-경로-벨만-포드Bellman-Ford-알고리즘>
* <https://deep-learning-study.tistory.com/587>
* <https://headf1rst.github.io/algorithm/bellmanford/>