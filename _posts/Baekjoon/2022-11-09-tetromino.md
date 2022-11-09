---
title: "[Baekjoon/🥈GoldⅣ] 14500: 테트로미노"
excerpt: "DFS"
categories:
  - Algorithm
tag:
  - algorithm
  - baekjoon
  - python
  - DFS

last_modifeid_at: 2022-11-09
toc: true
toc_sticky: true
search: true
---

## 🙋‍♀️ [테트로미노](https://www.acmicpc.net/problem/14500)

문제에 대한 자세한 설명은 위 페이지를 참고!!

## 🙋‍♀️ 입력
첫째 줄에 종이의 세로 크기 N과 가로 크기 M이 주어진다. (4 ≤ N, M ≤ 500)

둘째 줄부터 N개의 줄에 종이에 쓰여 있는 수가 주어진다. i번째 줄의 j번째 수는 위에서부터 i번째 칸, 왼쪽에서부터 j번째 칸에 쓰여 있는 수이다. 입력으로 주어지는 수는 1,000을 넘지 않는 자연수이다.

## 🙋‍♀️ 출력
첫째 줄에 테트로미노가 놓인 칸에 쓰인 수들의 합의 최댓값을 출력한다.

<BR>

## 👩‍💻 Algoritm: Brute Force

필자의 경우, 가능한 모든 경우를 직접 배열에 넣어 확인하는 방식으로 **브루트 포스**를 적용하였다.

```python
import sys

n, m = map(int, sys.stdin.readline().split())
answer = 0
graph = []

for _ in range(n):
  graph.append(list(map(int, sys.stdin.readline().split())))

dx = [(0, 1, 2, 3), (0, 0, 0, 0), (0, 0, 0, 1), (0, 1, 2, 2), (0, 1, 1, 1)
  , (0, 1, 1, 1), (0, 0, 1, 2), (0, -1, -1, -1), (0, -1, -1, -2), (0, 0, 1, 1), (0, 0, -1, -1)
    , (0, 1, 1, 2), (0, 1, 1, 2), (0, -1, 0, 1), (0, 0, 0, 1), (0, 0, 0, -1), (0, 0, 1, 1), (0, 0, 1, 2), (0, 0, -1, -2)]
dy = [(0, 0, 0, 0), (0, 1, 2, 3), (0, 1, 2, 2), (0, 0, 0, -1), (0, 0, 1, 2)
  , (0, 0, -1, -2), (0, 1, 1, 1), (0, 0, 1, 2), (0, 0, 1, 1), (0, 1, 1, 2), (0, 1, 1, 2)
  , (0, 0, 1, 1), (0, 0, 1, 0), (0, 1, 1, 1), (0, 1, 2, 1), (0, 1, 2, 1), (0, 1, 1, 0), (0, -1, -1, -1), (0, -1, -1, -1)]

for i in range(n):
  for j in range(m):
    for k in range(19):
      s = 0
      for l in range(4):
        ny = i+dy[k][l]
        nx = j+dx[k][l]

        if 0<=ny<n and 0<=nx<m:
          s += graph[ny][nx]
        else:
          break
      answer = max(answer, s)

print(answer)
```

하지만 이렇게 일일이 모든 경우의 수를 직접 구하는 방법 말고는 다른 방법은 없을까 생각하다 다른 분의 풀이를 보고 **dfs**를 사용할 수 있다는 점을 발견했다.

그렇다면 지금부터 dfs를 통해 풀이를 진행해보자!!

<br>

## 👩‍💻 Algorithm: dfs

아래 그림의 도형들의 회전, 대칭 시 만들어지는 모든 도형들은 **깊이 3인 dfs**를 통해서 구할 수 있다.

<img src = "https://drive.google.com/uc?id=1nTwgl4Z96qN-VLRPVDnG83gJu0K5NBuw" width = 500 height = 500>

그렇다면 그 이유는 무엇일까? 

1. 우선 위 도형들은 그림과 같이 **하나의 선으로 이을 수 있다.** 따라서 **dfs**를 사용할 수 있다.

2. **테트로미노**의 경우, **4개의 정사각형**으로 이루어졌다. 그렇기 때문에 깊이 3만큼을 이동해 총 4개의 정사각형으로 이루어진 모든 도형을 확인할 수 있게 된다.

그렇다면 나머지 한 도형의 경우는 어떻게 처리해야 할까?

<img src = "https://drive.google.com/uc?id=1oCVCMsRPrVQccYyrFyRar4317SAopRIa" width = 500 height = 500>

그림과 같이 해당 도형은 시작점으로부터 **상하좌우 중 세가지 요소만**을 고려하면 된다. 이는 위 dfs 과정과는 별도로 추가적으로 진행하면 된다.

그렇다면 아래의 코드를 통해서 조금 더 정확히 그 과정을 이해해보자!!

<br>

```python
import sys

n, m = map(int, sys.stdin.readline().split())
graph = [list(map(int, sys.stdin.readline().split())) for _ in range(n)]
visited = [[0]*m for _ in range(n)]

# 최대값
maxValue = 0
 
# 상하좌우를 나타내는 좌표
dy = [0, 0, -1, 1]
dx = [-1, 1, 0, 0]

def dfs(y, x, s, t):
  global maxValue

  if t == 4: # 시작점(1)에서 깊이가 3일 경우
    maxValue = max(maxValue, s)
    return

  for i in range(4):
    ny = y + dy[i]
    nx = x + dx[i]

    if 0<=ny<n and 0<=nx<m and visited[ny][nx] == 0:
      visited[ny][nx] = 1
      dfs(ny, nx, s+graph[ny][nx], t+1)
      visited[ny][nx] = 0

# ㅗ, ㅏ, ㅜ, ㅓ의 경우
def shapeHats(y, x):
  global maxValue

  for i in range(4):
    tmp = graph[y][x] # 시작지점으로 초기값 설정
    for j in range(3): # 상하좌우 중 세요소만 고려
      ny = y + dy[(i+j)%4]
      nx = x + dx[(i+j)%4]

      if not (0<=ny<n and 0<=nx<m):
        tmp = 0
        break
      tmp += graph[ny][nx]
    maxValue = max(maxValue, tmp)

# 모든 경우의 수 파악
for i in range(n):
  for j in range(m):
    visited[i][j] = 1
    dfs(i, j, graph[i][j], 1)
    visited[i][j] = 0

    shapeHats(i, j) # dfs와 별도로 추가 진행

print(maxValue)
```

<br>

## 🙋‍♀️ TIL

처음 모양이 제각각인 도형의 최댓값을 어떻게 구해야할지 감이 잘 오지 않았다.

하지만 문제에서 주목해야 할 부분은 도형의 모양들이 아니라 도형의 특징이었다. **하나의 선****으로 이어지기에 **dfs**를 적용할 수 있었고, 도형의 **구성 정사각형의 수**에 따라서 그 **깊이를 지정**할 수 있었다. 

dfs의 새로운 접근법에 대해 학습할 수 있었던 것 같다.

<br>

# 📝 참고 자료
* <https://cijbest.tistory.com/87>
* <https://www.acmicpc.net/problem/14500>