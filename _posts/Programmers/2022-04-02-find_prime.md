---
title: "[Programmers/ level2] 소수 찾기"
categories:
  - Programmers
tag:
 - python
 - programmers
 - level2
 - 에라토스테네스의 체

last_modifeid_at: 2022-04-02
toc: true
toc_sticky: true
search: true
---

<br>

# 👩문제
한자리 숫자가 적힌 종이 조각이 흩어져있습니다. 흩어진 종이 조각을 붙여 소수를 몇 개 만들 수 있는지 알아내려 합니다.

각 종이 조각에 적힌 숫자가 적힌 문자열 numbers가 주어졌을 때, 종이 조각으로 만들 수 있는 소수가 몇 개인지 return 하도록 solution 함수를 완성해주세요.

<br>

## 👩제한사항
* numbers는 길이 1 이상 7 이하인 문자열입니다.
* numbers는 0~9까지 숫자만으로 이루어져 있습니다.
* "013"은 0, 1, 3 숫자가 적힌 종이 조각이 흩어져있다는 의미입니다.

<br>

## 👩입출력 예

|numbers|return|
|-------|------|
|"17"|3|
|"011"|2|

🙄입출력 예 설명
* 예제 #1
  * [1, 7]으로는 소수 [7, 17, 71]를 만들 수 있습니다.

* 예제 #2
  * [0, 1, 1]으로는 소수 [11, 101]를 만들 수 있습니다.
  * 11과 011은 같은 숫자로 취급합니다.

<br>

## 🙋‍♀️내 풀이
1. 총 7개의 문자열이 존재할 수 있으므로 9999999까지 에라토스테네스의 체를 만들었다.(막무가내..ㅎㅎ)

2. **itertools.permutations**을 사용해 요소 개수를 **1-문자열 길이**까지 지정해 각 경우의 수가 소수인지를 판단하였다.

```
from itertools import permutations

#에라토스테네스 체 생성!
def getPrime():
    lst = [1 for i in range(10000000)]
    lst[1] = 0; lst[0] = 0
    for i in range(2, int(9999999**0.5) + 1):
        if lst[i] == 1:
            for j in range(i*2, 10000000, i):
                lst[j] = 0
    return lst      

def solution(numbers):
    answer = 0
    a = set()
    lst = getPrime()
    for i in range(len(numbers)):
        a |= set(map(int, map(''.join, permutations(list(numbers), i+1))))
    a -= set(range(2))
    for i in a:
        if lst[i] == 1:
            lst[i] = 3
            answer += 1
    return answer
```
🙄 시간이 무척 오래 걸리는 비효율적인 방법이다. <u>문자열로 만들 수 있는 수 중 가장 큰 수</u>까지 에라토스테네스의 체의 범위를 지정했으면 조금 더 좋았을 것 같다. 

<br>

## 🙋‍♀️best 풀이 : set 적용하기

```
from itertools import permutations

def solution(numbers):
    a = set()
    for i in range(1, len(numbers)+1):
        # '|='로 set update
        a |= set(map(int, map(''.join, permutations(list(numbers), i))))
    # '-='로 set 제거
    a -= set(range(0, 2))
    for i in range(2, int(max(a)**0.5)+1):
        a -= set(range(i*2, max(a)+1, i))
    return len(a)
```

🙄위 풀이의 방식은 아래와 같다.
* 문자열로 만들 수 있는 모든 수를 구한 후, **set()에 넣음으로써 중복을 제거한다.**
* set에서 에라토스테네스의 체를 적용하여 소수가 아닌 것을 제외시킨다.

<br>

## 🙋‍♀️다른 풀이 : 재귀 사용하기

```
primeSet = set()

def isPrime(num):
    if num in (0, 1):
        return False
    for i in range(2, num):
        if num % i == 0:
            return False
    return True

def makeCombi(str1, str2):
    if str1 != "":
        if isPrime(int(str1)):
            primeSet.add(int(str1))
          
    # 경우의 수를 구하는 방식이 새로움!
    for i in range(len(str2)):
        makeCombi(str1 + str2[i], str2[:i] + str2[i+1:])

def solution(numbers):
    makeCombi("", numbers)
    return len(primeSet)
```

🙄위 풀이에서 주목할 만한 점은 경우의 수를 구하는 방식이다!
<img src = "https://ifh.cc/g/3xr0pb.jpg" width = 600 height = 500>
* 기존 문자열(str1)과 **붙이지 않은 문자열(str2)의 i번째 문자열**을 붙이는 방식으로 모든 경우의 수를 구했다.

<br>

# 🙇‍♀️알게된 점
* 튜플도 join함수의 인자로 가능하다.
* permutations에 요소의 길이를 설정할 수 있다.
* set(range(?))처럼 for문을 돌리지 않고 값을 생성할 수 있다.
  * list(range(?))도 마찬가지이다.

<br>

## 📃 참고
* <https://programmers.co.kr/learn/courses/30/lessons/42839>
* <https://programmers.co.kr/learn/courses/30/lessons/42839/solution_groups?language=python3>