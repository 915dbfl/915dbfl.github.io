---
title: "[Programmers/ level3] 보석 쇼핑🙄"
excerpt: "투포인터 적용"
categories:
  - Programmers
tag:
 - python
 - programmers
 - level3
 - 투 포인터
 - 보석 쇼핑

last_modifeid_at: 2022-07-14
toc: true
toc_sticky: true
search: true
---

<br>


# 👩[문제](https://school.programmers.co.kr/learn/courses/30/lessons/67258)
[본 문제는 정확성과 효율성 테스트 각각 점수가 있는 문제입니다.]

개발자 출신으로 세계 최고의 갑부가 된 어피치는 스트레스를 받을 때면 이를 풀기 위해 오프라인 매장에 쇼핑을 하러 가곤 합니다.
어피치는 쇼핑을 할 때면 매장 진열대의 특정 범위의 물건들을 모두 싹쓸이 구매하는 습관이 있습니다.
어느 날 스트레스를 풀기 위해 보석 매장에 쇼핑을 하러 간 어피치는 이전처럼 진열대의 특정 범위의 보석을 모두 구매하되 특별히 아래 목적을 달성하고 싶었습니다.


`진열된 모든 종류의 보석을 적어도 1개 이상 포함하는 가장 짧은 구간을 찾아서 구매`

예를 들어 아래 진열대는 4종류의 보석(RUBY, DIA, EMERALD, SAPPHIRE) 8개가 진열된 예시입니다.

|진열대 번호| 1|2|3|4|5|6|7|8|
|-----------|--|-|-|-|-|-|-|-|
|보석 이름|DIA|RUBY|RUBY|DIA|DIA|EMERALD|SAPPHIRE|DIA|


진열대의 3번부터 7번까지 5개의 보석을 구매하면 모든 종류의 보석을 적어도 하나 이상씩 포함하게 됩니다.

진열대의 3, 4, 6, 7번의 보석만 구매하는 것은 중간에 특정 구간(5번)이 빠지게 되므로 어피치의 쇼핑 습관에 맞지 않습니다.

진열대 번호 순서대로 보석들의 이름이 저장된 배열 gems가 매개변수로 주어집니다. 이때 모든 보석을 하나 이상 포함하는 가장 짧은 구간을 찾아서 return 하도록 solution 함수를 완성해주세요.
가장 짧은 구간의 시작 진열대 번호와 끝 진열대 번호를 차례대로 배열에 담아서 return 하도록 하며, 만약 가장 짧은 구간이 여러 개라면 시작 진열대 번호가 가장 작은 구간을 return 합니다.

## 👩제한사항
* gems 배열의 크기는 1 이상 100,000 이하입니다.
    * gems 배열의 각 원소는 진열대에 나열된 보석을 나타냅니다.
    * gems 배열에는 1번 진열대부터 진열대 번호 순서대로 보석이름이 차례대로 저장되어 있습니다.
    * gems 배열의 각 원소는 길이가 1 이상 10 이하인 알파벳 대문자로만 구성된 문자열입니다.

<br>

## 👩입출력 예

|gems|result|
|----|------|
|["DIA", "RUBY", "RUBY", "DIA", "DIA", "EMERALD", "SAPPHIRE", "DIA"]|[3, 7]|
|["AA", "AB", "AC", "AA", "AC"]|[1, 3]|
|["XYZ", "XYZ", "XYZ"]|[1, 1]|
|["ZZZ", "YYY", "NNNN", "YYY", "BBB"]|[1, 5]|

* ~~입출력에 대한 설명은 [해당 사이트](https://school.programmers.co.kr/learn/courses/30/lessons/67258)를 확인해주세요!~~

<br>

## 🙋‍♀️접근 방법
* 나의 접근 방법(오답)
    * 처음 나는 각 보석마다 진열된 시작 위치와 끝위치를 구해 무조건 포함되는 구간을 구하는 방식으로 접근했다.

        |진열대 번호| 1|2|3|4|5|6|7|8|
        |-----------|--|-|-|-|-|-|-|-|
        |보석 이름|DIA|RUBY|RUBY|DIA|DIA|EMERALD|SAPPHIRE|DIA|

    * 예를 들어 위 예시에서 EMERALD와 SAPPHIRE는 각각 1개밖에 존재하지 않으므로 해당 위치는 무조건 포함되어야 한다.

    * 이렇게 접근했을 때 문제점은 gems 배열을 모두 돌아 각 보석의 위치를 파악해야 했을 뿐 아니라 각 보석의 시작, 끝 위치를 담은 배열(str, end)에서 최대값, 최소값을 비교해 구한 구간이 원래 gems 배열의 크기보다 크게 줄어들지 않았다. 결국 gems 배열 전체를 완전 탐색하는 결과와 비슷한 O(N^2)의 복잡도를 가지게 된다.
    
-------

* 접근 방법(해답) : `🔔투 포인터🔔`
    1. **시작하는 범위(left)**와 **끝 범위(right)**를 가리키는 **두 개의 포인터**를 두고, 모두 진열대 **처음**을 가리키게 한다.
    2. 구간 안의 보석의 종류를 파악한다.
        * ~~이때 배열 슬라이싱을 이용할 경우, 슬라이싱 범위만큼의 시간복잡도를 가지기 때문에 종류를 파악하기 위해 보석의 종류를 key로 갖는 **딕셔너리**를 사용한다.~~
        * 2-1. 범위 안에 있는 보석의 종류가 전체 보석의 종류보다 **같을** 경우, **left 값을 1씩 증가**하며 해당 right에서의 최소 구간(best)을 구해 후보값(result)에 추가시킨다.
        * 2-2. 범위 안에 있는 보석의 종류가 전체 보석의 종류보다 **적을** 경우, **right를 1 증가**시킨다.
        * 2-3. **left, right 값이 배열을 넘기 전**까지 2번 과정을 **반복**한다.
    3. 후보값 중 구간의 크기가 가장 짧은 값이 정답이 된다. (구간의 크기가 동일할 경우, 시작이 빠른 구간이 정답이 된다.)


<br>

## 🙋‍♀️코드 

```python
from collections import defaultdict

def solution(gems):
    num = len(set(gems))
    result = []
    dic = defaultdict(int)

    left = 0
    best = 0 # 최소 구간인지를 나타내는 값, 0 또는 1
    for right in range(len(gems)):
        dic[gems[right]] += 1 # right+1로 추가되는 보석 포함
        right += 1
        while len(dic) == num: # 구간의 보석 종류가 전체 보석 종류와 동일할 때
            dic[gems[left]] -= 1 # left+1을 위해 해당 위치 보석 제외
            if dic[gems[left]] == 0: # 없는 보석의 경우 종류에서 제거
                del dic[gems[left]]
            left += 1 # left+1 진행
            best = 1 # 최소 구간임을 표시
        if best:
            result.append([left, right]) # right에서의 최소 구간을 후보에 추가
            best = 0

    return sorted(result, key = lambda x: (x[1]-x[0], x[0]))[0]
```
<br>

## 👩‍💻배운 점
* `collections.defaultdict()`: 딕셔너리 **기본값/초기값** 설정
    * defaultdict(int): 초기 값 0
    * defaultdict(list): 초기 값 []
    * defaultdict(set): 초기 값 ()
* 배열의 **구간마다** 특정 계산을 진행해야 할 때 **투 포인터**를 사용하는 것이 효과적이다.

<BR>

🙇‍♀️풀이에 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃 참고
* 문제 관련 참고자료
    * <https://school.programmers.co.kr/learn/courses/30/lessons/67258>
    * <https://jennnn.tistory.com/793>
    * <https://haeseok.medium.com/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8%EC%8A%A4-%EB%B3%B4%EC%84%9D%EC%87%BC%ED%95%91-a901de0ce34>
    * <https://school.programmers.co.kr/questions/28562>
* defaultdict()
    * <https://dongdongfather.tistory.com/69>