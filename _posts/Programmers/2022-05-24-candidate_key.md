---
title: "[Programmers/ level2] 후보키"
excerpt: "zip(*iterable)"
categories:
  - Programmers
tag:
 - python
 - programmers
 - level2
 - 후보키

last_modifeid_at: 2022-05-24
toc: true
toc_sticky: true
search: true
---

<br>

# 👩문제

<img src = "https://ifh.cc/g/jZnGHN.png" width = 500 height = 500>

<img src = "https://ifh.cc/g/OLzXQk.jpg" width = 500 height = 600>
<br>

## 👩제한사항
* relation은 2차원 문자열 배열이다.
* relation의 컬럼(column)의 길이는 1 이상 8 이하이며, 각각의 컬럼은 릴레이션의 속성을 나타낸다.
* relation의 로우(row)의 길이는 1 이상 20 이하이며, 각각의 로우는 릴레이션의 튜플을 나타낸다.
* relation의 모든 문자열의 길이는 1 이상 8 이하이며, 알파벳 소문자와 숫자로만 이루어져 있다.
* relation의 모든 튜플은 유일하게 식별 가능하다.(즉, 중복되는 튜플은 없다.)

<br>

## 👩입출력 예

|relation|result|
|------|-----|
|[["100","ryan","music","2"],["200","apeach","math","2"],["300","tube","computer","3"],["400","con","computer","4"],["500","muzi","music","3"],["600","apeach","music","2"]]|2|


<br>

## 🙋‍♀️내 풀이: heap 사용
combinations을 활용하여 key가 될 수 있는 모든 조합을 구하였다. 

> 유일성 확인

* 그렇게 구한 조합으로 key를 만들었을 경우, **set()**을 통해서 유일성이 만족되는 지 확인한다. 


> 최소성 확인

* 키가 되는 조합들은 별도의 리스트에 저장한다.
* 새로운 조합을 확인할 경우, 리스트에 해당 조합이 존재하는지 확인한다.

    <br>

    🔔여기서 주의할 점🔔
    * 단순히 리스트의 조합들이 새로운 조합에 포함되는지만을 확인하면 안된다.
    * 예를 들어 리스트 속 조합 중 하나가 abc, 새로운 조합은 abdc라 해보자.
        * 새로운 조합에 **abc가 포함되므로** 이는 최소성을 만족하지 않는다.
        * 만약 리스트 조합이 새로운 조합에 **포함되는지 확인한다면** 최소성을 만족하는 것이다.
        * abc각 요소가 새로운 조합에 포함되는지 **각각 확인해야** 최소성을 만족하지 않는 것을 확인할 수 있다.

    
```
from itertools import combinations

def solution(relation):
    answer = 0
    lstA = []
    lst = [[] for _ in range(len(relation[0]))]
    for case in relation:
        for i, val in enumerate(case):
            lst[i].append(val) #👩열을 기준으로 리스트를 만든다.
    nums = [str(i) for i in range(len(lst))]
    cnt = 1
    while cnt <= len(nums):
        for j in combinations(nums, cnt):
            for k in lstA: #👩최소성 판단
                for c in k:
                    if c not in j:
                        break
                else:
                    break        
            else:
                tmp = [lst[int(case)] for case in j]
                chk = set(c for c in zip(*tmp)) #👩유일성 판단
                if len(chk) == len(relation):
                    answer += 1
                    lstA.append(j)
        cnt += 1
    return answer
```

### 🙄 zip 함수: 데이터 묶기
* iterable 자료형의 **각 요소들을 묶어** 요소의 개수만큼 새로운 iterable 자료형을 생성한다.
* 이때 iterable 자료형들은 **요소의 개수가 동일**해야 한다.
* **zip(*iterable)**의 경우, 해당 **자료형 안** <u>iterable 자료형의 각 요소</u>들을 묶게 된다.

위의 코드를 통해서 조금 더 자세히 살펴보자!

```
combi = [['ryan', 'apeach', 'tube', 'con', 'muzi', 'apeach'], ['music', 'math', 'computer', 'computer', 'music', 'music']]
tmp = set(c for c in zip(*combi))

print(tmp) # {('apeach', 'music'), ('muzi', 'music'), ('tube', 'computer'), ('ryan', 'music'), ('apeach', 'math'), ('con', 'computer')}
['0', '12']

```

<BR>

## 🙋‍♀️다른 풀이: 비트 연산 사용

```
def solution(relation):
    answer = []
    for i in range(1, 1<<len(relation[0])): #👩 가능한 모든 조합을 확인한다.
        tmp_set = set() 
        for j in range(len(relation)):
            tmp = []
            for k in range(len(relation[0])):
                if i & (1 << k): #👩 해당 조합에 들어가는 키인지 판단한다.
                    tmp.append(str(relation[j][k]))
            tmp_set.add(tuple(tmp))
        if len(tmp_set) == len(relation): #👩 유일성을 판단한다.
            chk = True
            for c in answer:
                if (c & i) == c: #👩 최소성을 판단한다.
                    chk = False
                    break
            if chk:
                answer.append(i)               
    return len(answer)
```

🙄 해당 풀이를 이해하는데 시간이 좀 걸렸다. 다 이해하고 나니 정말 새로운 접근 방식인 것 같아서 정리해보고자 한다.

* ☝️ 후보키 가능한 **모든 조합**을 combinations을 사용해 구하지 않고 **비트**를 사용해 구했다.
    * 즉, 해당 풀이에서는 **n개의 컬럼을 조합하는 모든 경우의 수**를 **n개의 비트**를 통해서 표현하였다.
    * 컬럼이 세개일 때, <u>001이 a컬럼을 포함하는 경우</u>, <u>011이 a와 b</u>, <u>111이 a,b,c 세 컬럼을 모두 포함하는 경우</u>가 된다.


* ✌️ 해당 조합에 **포함되는 컬럼**을 **'if i & (1 << k)'**을 통해서 확인한다.
    * k = 3일 때 '1 << k'를 할 경우 1000이 된다. 해당 비트를 i와 &(AND)연산을 진행해 포함된 컬럼인지를 확인한다. 


* 👌 포함된 컬럼들의 값을 tmp에 저장하고 이를 다시 tmp_set에 저장한 후, 그 길이를 전체 학생 수인 relation의 길이와 비교해 **유일성**을 판단한다.


* ✋ 기존 조합이 새로운 조합에 포함되었는지를 판단함으로써 **최소성**을 판단한다. 

<br>

👩 해당 풀이를 풀이하면서 조합 문제를 **비트를 활용하는 방식**으로 접근할 수 있다는 점을 새롭게 배울 수 있었다!!

<br>

🙇‍♀️풀이에 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃 참고
* <https://ooyoung.tistory.com/60>
* <https://wikidocs.net/32>
* <https://dojang.io/mod/page/view.php?id=2315>
* <https://programmers.co.kr/learn/courses/30/lessons/42890/solution_groups?language=python3>
* <https://programmers.co.kr/learn/courses/30/lessons/42890?language=python3>