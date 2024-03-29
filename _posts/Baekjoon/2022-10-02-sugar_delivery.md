---
title: "[Baekjoon/🥈SilverⅣ] 2839: 설탕 배달"
excerpt: "greedy/dp"
categories:
  - Algorithm
tag:
  - algorithm
  - baekjoon
  - python
  - greedy
  - dp

last_modifeid_at: 2022-10-02
toc: true
toc_sticky: true
search: true
---

## 🙋‍♀️ [설탕 배달](https://www.acmicpc.net/problem/2839)

상근이는 요즘 설탕공장에서 설탕을 배달하고 있다. 상근이는 지금 사탕가게에 설탕을 정확하게 N킬로그램을 배달해야 한다. 설탕공장에서 만드는 설탕은 봉지에 담겨져 있다. 봉지는 3킬로그램 봉지와 5킬로그램 봉지가 있다.

상근이는 귀찮기 때문에, 최대한 적은 봉지를 들고 가려고 한다. 예를 들어, 18킬로그램 설탕을 배달해야 할 때, 3킬로그램 봉지 6개를 가져가도 되지만, 5킬로그램 3개와 3킬로그램 1개를 배달하면, 더 적은 개수의 봉지를 배달할 수 있다.

상근이가 설탕을 정확하게 N킬로그램 배달해야 할 때, 봉지 몇 개를 가져가면 되는지 그 수를 구하는 프로그램을 작성하시오.

<br>

## 🙋‍♀️ 입력
첫째 줄에 N이 주어진다. (3 ≤ N ≤ 5000)

## 🙋‍♀️ 출력
상근이가 배달하는 봉지의 최소 개수를 출력한다. 만약, 정확하게 N킬로그램을 만들 수 없다면 -1을 출력한다.

<br>

## 👩‍💻 Algoritm: greedy

필자는 가장 먼저 **탐욕기법(greedy)**을 적용해 알고리즘을 작성했다.

봉지의 최소 개수를 구하는 것이므로 **5킬로그램의 봉지**를 **최대한 많이** 쓰게 해야 한다.

### 👩 알고리즘1: 5킬로그램 봉지 사용의 최대를 구한 후 줄여 나간다.

```python
N = int(input())

five = N//5 # 5킬로그램 봉지를 최대로 쓸 경우

for i in range(five, -1, -1): # 5킬로그램 봉지를 하나씩 줄여간다.
  three = (N-(i*5))%3
  if three == 0:
    print(i+((N-(i*5))//3))
    break
else:
  print(-1)
```

<br>

다른 사람들 풀이를 보다 보니 공통적인 방식으로 그리디를 적용하고 있어 해당 문제 풀이도 정리해본다.

### 👩 알고리즘2: 5로 나누어 떨어지나? 3으로 나누어 떨어지나?

풀이 과정을 설명하면 다음과 같다!
* <U>N이 5로 나누어 떨어진다면</U> **N//5**로 봉지수를 구한 후 출력한다.
* N이 5로 나누어 떨이지지 않는다면 다음의 과정을 거친다.
  * **N-3**을 하여 **3킬로그램 봉지를 하나** 사용한다.
  * <U>N이 0보다 작을 경우</U>, 정확하게 N킬로그램을 만들 수 없는 것이므로 **-1**을 출력한다.

```python
N = int(input())
cnt = 0

while 1:
  if N % 5 != 0:
    cnt += 1
    N -= 3
  elif N % 5 == 0:
    print(cnt + N//5)
    break
  
  if N < 0:
    print(-1)
    break
```
<br>

## 👩‍💻 Algoritm: dp

해당 문제를 dp를 적용해 풀이할 수도 있다.


나는 dp를 적용해볼 생각을 아예 안해봤다...ㅎㅎㅎ 정리를 통해서 그 과정을 다시 한 번 익히자!!!

<br>

그렇다면 이 문제에 dp가 어떻게 적용되는 것일까???🙄

* 각각 N킬로그램일때 값은 과연 어떻게 얻어지는 것일까?
  * **[N-5] 킬로그램**일 때 최소 봉지에다 **5킬로그램 봉지 하나!**
  * **[N-3] 킬로그램**일 때 최소 봉지에다 **3킬로그램 봉지 하나!**
  * 위 **두 값 중 최소 값 + 1** => 점화식: `min(dp[n-3], dp[n-5])`

그렇다! N킬로그램일 때 최소 봉지는 오직 **[N-5], [N-3]킬로그램일 때의 봉지 수**에 의해서 결정된다! 따라서 우리는 dp를 적용할 수 있는 것이다!

<br>

### 👩 알고리즘1: dp 적용!

```python
N = int(input())
dp = [-1 for _ in range(N+1)]

if N >= 3:
  dp[3] = 1
if N >= 5:
  dp[5] = 1

for i in range(6, N+1):
  if dp[i-5] == -1 and dp[i-3] == -1:
    continue
  elif dp[i-5] == -1:
    dp[i] = dp[i-3] +1
  elif dp[i-3] == -1:
    dp[i] = dp[i-5] +1
  else:
    dp[i] = min(dp[i-5], dp[i-3]) +1

print(dp[N])
```
* 초기값은 `-1`로 지정한다.
* dp[n-5], dp[n-3]이 **모두 -1**일 경우, 정확하게 n킬로그램을 **맞출 수 없다**는 뜻이므로 초기값 -1이 유지된다.
* dp[n-5], dp[n-3] 중 하나라도 -1이 아니라면 아닌 킬로그램에 봉지 하나를 추가하여 n킬로그램을 만들 수 있다.
* dp[n-5], dp[n-3] 모두 -1이 아니라면 **둘 중 최소값**을 선택해 봉지 하나를 추가하여 n킬로그램을 만들 수 있다.

<br>

하지만 여기서 그칠 것이 아니라 우리는 하나 더 생각해볼 수 있다!
여기서 **공간복잡도는 O(n)**이다. 그렇지만 n일 때 필요한 부분은 `n-5`까지만 필요하고 그 이전 값들은 필요하지 않는다!!

⭐즉! `0-5`까지만 있으면 된다는 뜻이다!!⭐

<br>

### 👩 알고리즘2: 공간 복잡도를 줄이기

```python
N = int(input())
dp = [-1 for _ in range(6)]

if N >= 3:
  dp[3] = 1
if N >= 5:
  dp[5] = 1

for i in range(6, N+1):
  if dp[(i-5)%6] == -1 and dp[(i-3)%6] == -1:
    continue
  elif dp[(i-5)%6] == -1:
    dp[i%6] = dp[(i-3)%6] +1
  elif dp[(i-3)%6] == -1:
    dp[i%6] = dp[(i-5)%6] +1
  else:
    dp[i%6] = min(dp[(i-5)%6], dp[(i-3)%6]) +1

print(dp[N%6])
```

<br>

# 📝 참고 자료
* <https://udou.tistory.com/entry/%EB%B0%B1%EC%A4%80-2839-%EC%84%A4%ED%83%95-%EB%B0%B0%EB%8B%AC-Python>
* <https://myjamong.tistory.com/291>