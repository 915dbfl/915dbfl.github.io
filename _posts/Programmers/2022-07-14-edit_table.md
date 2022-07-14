---
title: "[Programmers/ level3] 표 편집🙄"
excerpt: "삽입, 삭제, 탐색의 반복"
categories:
  - Programmers
tag:
 - python
 - programmers
 - level3
 - 표 편집
 - 파이썬 링크드리스트

last_modifeid_at: 2022-07-14
toc: true
toc_sticky: true
search: true
---

<br>


# 👩[문제](https://school.programmers.co.kr/learn/courses/30/lessons/81303?language=python3)
[본 문제는 정확성과 효율성 테스트 각각 점수가 있는 문제입니다.]

업무용 소프트웨어를 개발하는 니니즈웍스의 인턴인 앙몬드는 명령어 기반으로 표의 행을 선택, 삭제, 복구하는 프로그램을 작성하는 과제를 맡았습니다. 세부 요구 사항은 다음과 같습니다.

|행 번호|이름|
|---|--|
|0|무지|
|1|콘|
|2|어피치|
|3|제이지|
|4|프로도|
|5|네오|
|6|튜브|
|7|라이언|

위 그림에서 파란색으로 칠해진 칸은 현재 선택된 행을 나타냅니다. 단, 한 번에 한 행만 선택할 수 있으며, 표의 범위(0행 ~ 마지막 행)를 벗어날 수 없습니다. 이때, 다음과 같은 명령어를 이용하여 표를 편집합니다.

* "U X": 현재 선택된 행에서 X칸 위에 있는 행을 선택합니다.
* "D X": 현재 선택된 행에서 X칸 아래에 있는 행을 선택합니다.
* "C" : 현재 선택된 행을 삭제한 후, 바로 아래 행을 선택합니다. 단, 삭제된 행이 가장 마지막 행인 경우 바로 윗 행을 선택합니다.
* "Z" : 가장 최근에 삭제된 행을 원래대로 복구합니다. **단, 현재 선택된 행은 바뀌지 않습니다.**

<br>

## 👩제한사항
* 제한사항
    * 5 ≤ n ≤ 1,000,000
    * 0 ≤ k < n
    * 1 ≤ cmd의 원소 개수 ≤ 200,000
        * cmd의 각 원소는 "U X", "D X", "C", "Z" 중 하나입니다.
        * X는 1 이상 300,000 이하인 자연수이며 0으로 시작하지 않습니다.
        * X가 나타내는 자연수에 ',' 는 주어지지 않습니다. 예를 들어 123,456의 경우 123456으로 주어집니다.
        * cmd에 등장하는 모든 X들의 값을 합친 결과가 1,000,000 이하인 경우만 입력으로 주어집니다.
        * 표의 모든 행을 제거하여, 행이 하나도 남지 않는 경우는 입력으로 주어지지 않습니다.
        * 본문에서 각 행이 제거되고 복구되는 과정을 보다 자연스럽게 보이기 위해 "이름" 열을 사용하였으나, "이름"열의 내용이 실제 문제를 푸는 과정에 필요하지는 않습니다. "이름"열에는 서로 다른 이름들이 중복없이 채워져 있다고 가정하고 문제를 해결해 주세요.
    * 표의 범위를 벗어나는 이동은 입력으로 주어지지 않습니다.
    * 원래대로 복구할 행이 없을 때(즉, 삭제된 행이 없을 때) "Z"가 명령어로 주어지는 경우는 없습니다.
    * 정답은 표의 0행부터 n - 1행까지에 해당되는 O, X를 순서대로 이어붙인 문자열 형태로 return 해주세요.
* 정확성 테스트 케이스 제한 사항
    * 5 ≤ n ≤ 1,000
    * 1 ≤ cmd의 원소 개수 ≤ 1,000
* 효율성 테스트 케이스 제한 사항
    * 주어진 조건 외 추가 제한사항 없습니다.

<br>

## 👩입출력 예

|n|k|cmd|result|
|-|-|--|-----|
|8|2|["D 2","C","U 3","C","D 4","C","U 2","Z","Z"]|"OOOOXOOO"|
|8|2|["D 2","C","U 3","C","D 4","C","U 2","Z","Z","U 1","C"]|"OOXOXOOO"|


* ~~입출력에 대한 설명은 [해당 사이트](https://school.programmers.co.kr/learn/courses/30/lessons/81303?language=python3)를 확인해주세요!~~

<br>

## 👩제한시간 안내
* 정확성 테스트: 10초
* 효율성 테스트: 언어별로 작성된 정답 코드의 실행 시간의 적정 배수

<br>

## 🙋‍♀️접근 방법
해당 문제에서의 핵심은 **특정 자료구조에 대한 삽입, 삭제, 탐색의 반복**이다.

딱히 어려운 부분은 없으나 효율성 테스트가 존재하기 때문에 이에 적절한 자료구조를 선택해 문제를 해결할 수 있어야 한다.초기 필자는 **리스트**를 통해서 삽입, 삭제, 검색을 진행하여 다음과 같은 코드를 작성하였다.


### 🙋‍♀️코드: 리스트
```python
def solution(n, k, cmd):
    answer, lst = [], []
    for i in range(n):
        answer.append("X")
        lst.append(i)
    storage = []
    
    for str in cmd:
        if str == "C":
            storage.append((k, lst[k]))
            del lst[k]
            if k == len(lst):
                k-=1
        elif str == "Z":
            idx, val = storage.pop()
            lst.insert(idx, val)
            if idx <= k:
                k += 1
        else:
            c, a = str.split(" ")
            if c == "U":
                k -= int(a)
            elif c == "D":
                k += int(a)
    for i in lst:
        answer[i] = "O"
    return "".join(answer)
```
리스트을 사용할 경우, 각 연산에 대한 시간복잡도를 살펴보자!
* C: 인덱스를 이용한 리스트 요소 **삭제**는 **O(N)**의 복잡도를 갖게 된다.
* Z: **insert()**를 통해 리스트 특정 위치에 값을 삽입하는 것은 **O(N)**의 복잡도를 갖는다.
* U, D: up, down한 인덱스를 구해 접근하므로 O(1)이다.

따라서 복잡도는 최악의 경우 **O(N*len(cmd))**이므로 효율성 테스트에 통과하기에는 무리가 있다.

<BR>

여기서 적용할 수 있는 자료구조가 **더블 링크드 리스트**이다!!

<BR>


## 🔔링크드리스트

1. 삽입/삭제: 링크드리스트의 경우, **삽입/삭제 시** 인접 노드의 레퍼런스만 변경해주면 되기 때문에 **O(1)**의 시간복잡도를 갖게 된다.

2. 탐색
그렇다면 탐색의 경우 어떨까? 이는 어떤 링크드리스트냐에 따라서 달라진다!
* [**싱글 링크드 리스트**](https://underflow101.tistory.com/3): 인덱스를 통해 탐색한다면 배열 탐색처럼 가장 앞 노드부터 탐색을 진행해야 하기 때문에 **O(N)**의 시간복잡도를 갖는다.
* [**더블 링크드 리스트**](https://underflow101.tistory.com/4): 인덱스로 탐색을 한다면 **리스트의 크기와 탐색하고자 하는 index의 크기를 비교**하여 head, tail 중 가까운 곳에서 탐색을 시작한다.
    * 최대 **O(N/2)**의 복잡도를 갖는다.

👩 본 문제를 풀기 위해 링크드리스트를 클래스로 구현할 수도 있지만 **딕셔너리**에 링크드리스트의 특징을 적용하여 탐색을 보다 빠르게 진행할 수 있도록 하고자 한다!

<br>

## 🙋‍♀️코드: 링크드리스트 BY 딕셔너리

```python
def solution(n, k, cmd):
    table = {i: [i-1, i+1] for i in range(n)} # 더블링크드리스트처럼 양방향 노드에 대한 값을 가지고 있는다.
    table[0][0] = None
    table[n-1][1] = None
    answer = ["O"]*n
    storage = []
    cur = k
    
    for c in cmd:
        if c == "C": # 표에서 cur행을 삭제하고 다음 행을 가리키게 된다. 만약 cur이 마지막 행일 경우, 그 전 노드를 가르킨다.
            answer[cur] = "X"
            pre, next = table[cur]
            storage.append([pre, cur, next])
            if next == None: # 마지막 노드냐에 따라 cur을 업데이트한다.
                cur = pre
            else:
                cur = next
            if pre == None: # 인접 노드의 값들을 변경한다.
                table[next][0] = None
            elif next == None:
                table[pre][1] = None
            else:
                table[pre][1] = next
                table[next][0] = pre
        elif c == "Z":
            pre, idx, next = storage.pop() # 최근 삭제한 행에 대한 값을 가져온다.
            answer[idx] = "O"
            if pre == None: # 인접 노드의 값을 변경한다.
                table[next][0] = idx
            elif next == None:
                table[pre][1] = idx
            else:
                table[pre][1] = idx
                table[next][0] = idx 
        else:
            c1, c2 = c.split(" ")
            c2 = int(c2)
            if c1 == "U":
                for _ in range(c2):
                    cur = table[cur][0]
            else:
                for _ in range(c2):
                    cur = table[cur][1]
                    
    return "".join(answer)
```
~~코드만 길 뿐 코드는 쉽게 이해할 수 있을 겻이다!~~

<br>

그렇다면 정석대로 클래스를 통해 링크드리스트를 구현해 문제를 해결한 코드도 한번 살펴보자!!

<BR>

## 🙋‍♀️코드: 링크드리스트
```python
class DoubleLinkedList:
    class Node:
        def __init__(self, num, pre):
            self.pre = pre
            self.num = num
            self.next = None
            
    def __init__(self, num, start):
        self.root = self.Node(0, None) # 없는 노드를 탐색하기 위해 root값 저장
        self.current = None
        self.stack = []
        temp = self.root
        for i in range(1, num):
            new_node = self.Node(i, temp)
            temp.next = new_node
            if i == start:
                self.current = new_node
            temp = new_node
    
    def up(self, num):
        for _ in range(num):
            self.current = self.current.pre
            
    def down(self, num):
        for _ in range(num):
            self.current = self.current.next
            
    def remove(self):
        remove_node = self.current
        self.stack.append(remove_node)
        if remove_node.next:
            self.current = remove_node.next
            self.current.pre = remove_node.pre
            if remove_node == self.root:
                self.root = remove_node.next
            if remove_node.pre:
                remove_node.pre.next = self.current
        else:
            self.current = remove_node.pre
            self.current.next = None
            
    def recover(self):
        recover_node = self.stack.pop()
        if recover_node.pre:
            recover_node.pre.next = recover_node
        if recover_node.next:
            recover_node.next.pre = recover_node
            if recover_node.next == self.root:
                self.root = recover_node
    
    def get_root(self):
        return self.root

            
def solution(n, k, cmd):
    table = DoubleLinkedList(n, k)
    for c in cmd:
        if c == "C":
            table.remove()
        elif c == "Z":
            table.recover()
        else:
            c1, c2 = c.split(" ")
            c2 = int(c2)
            if c1 == "U":
                table.up(c2)
            else:
                table.down(c2)
    node = table.get_root()
    result = ["X"]*n
    while node:
        result[node.num] = "O"
        node = node.next
    return "".join(result)
```

<BR>

## 👩‍💻배운 점
* 삽입, 삭제, 탐색이 반복되는 문제에 대해서는 효율성을 따져 그에 맞는 자료구조를 선택해야 한다. 단순히, 구현만이 쉬운 것을 택한다면 효율적인 코드를 작성할 수 없다.

* 링크드리스트의 모든 노드에서는 pre, num, next값을 각각 가지고 있어야 한다. 이러한 상황같이 클래스를 이용한 캡슐화를 잘 사용한다면 효율적으로 코드를 작성할 수 있다.

<BR>

🙇‍♀️풀이에 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃 참고
* 링크드리스트
    * <https://underflow101.tistory.com/4>
    * <https://underflow101.tistory.com/3>
* 문제 관련 참고자료
    * <https://kjhoon0330.tistory.com/entry/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8%EC%8A%A4-%ED%91%9C-%ED%8E%B8%EC%A7%91-Python>
    * <https://school.programmers.co.kr/learn/courses/30/lessons/81303?language=python3>
* 각 자료구조의 삽입/삭제/탐색 시간복잡도
    * <https://carrido-hobbies-well-being.tistory.com/111>
* 파이썬 클래스
    * <https://wikidocs.net/28>