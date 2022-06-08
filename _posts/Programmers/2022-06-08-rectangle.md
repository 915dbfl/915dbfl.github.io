---
title: "[Programmers/ level2] 멀쩡한 사각형"
excerpt: "최대공약수, 최소공배수"
categories:
  - Programmers
tag:
 - python
 - programmers
 - level2
 - gcd
 - lcm
 - 멀쩡한 사각형

last_modifeid_at: 2022-06-08
toc: true
toc_sticky: true
search: true
---

<br>

# 👩문제

가로 길이가 Wcm, 세로 길이가 Hcm인 직사각형 종이가 있습니다. 종이에는 가로, 세로 방향과 평행하게 격자 형태로 선이 그어져 있으며, 모든 격자칸은 1cm x 1cm 크기입니다. 이 종이를 격자 선을 따라 1cm × 1cm의 정사각형으로 잘라 사용할 예정이었는데, 누군가가 이 종이를 대각선 꼭지점 2개를 잇는 방향으로 잘라 놓았습니다. 그러므로 현재 직사각형 종이는 크기가 같은 직각삼각형 2개로 나누어진 상태입니다. 새로운 종이를 구할 수 없는 상태이기 때문에, 이 종이에서 원래 종이의 가로, 세로 방향과 평행하게 1cm × 1cm로 잘라 사용할 수 있는 만큼만 사용하기로 하였습니다.
가로의 길이 W와 세로의 길이 H가 주어질 때, 사용할 수 있는 정사각형의 개수를 구하는 solution 함수를 완성해 주세요.

<br>

## 👩제한사항
* W, H : 1억 이하의 자연수

<br>

## 👩입출력 예

|W|H|result|
|------|-----|--|
|8|12|80|

###  👩입출력 예 설명
* 가로가 8, 세로가 12인 직사각형을 대각선 방향으로 자르면 총 16개 정사각형을 사용할 수 없게 됩니다. 원래 직사각형에서는 96개의 정사각형을 만들 수 있었으므로, 96 - 16 = 80 을 반환합니다.

<img src = "https://ifh.cc/g/NOhgJ4.png" width = 300 height = 400>

<br>

## 🙋‍♀️접근 방법

### 👩직선 방정식 이용1

가로가 w, 세로가 h인 사각형을 반으로 자르는 **직선 방정식**은 **'(-h/w)*i+h'**이다.

이를 통해서 **1부터 w까지 1씩 증가하며** 포함되는 사각형의 개수를 계산하여 최종적으로 2배를 함으로써 전체 정사각형을 구할 수 있었다. 

코드를 통해서 그 방식을 조금 더 자세히 살펴보자!

```
import math
def solution(w,h):
    answer = 0
    for i in range(1, w+1):
        answer += math.trunc((-h/w)*i+h)*2
    return answer

```
* 만약 직선 위 점의 y 값이 10.5이며 멀쩡한 정사각형은 총 10개가 된다. 따라서 **math.trunc**를 사용해 **버림**을 적용했다.

하지만 이 방식은 1부터 w까지 모두 확인해야 한다. 결국 시간초과...ㅎ

### 👩최대공약수 이용

<img src = "https://ifh.cc/g/aC1foC.jpg" width = 300 height = 400>

w, h의 **최대공약수**는 **직선에 지나는 점의 개수(=노란 사각형의 개수)**이다.

w = 8, h =12인 경우를 살펴보자. 이 둘의 최대공약수는 4이다.
따라서 반복되는 작은 사각형(위의 노란 사각형)은 총 4개가 존재한다.

여기서 핵심은 작은 <u>사각형이 반복될 때마다</u> 멀쩡한 사각형의 개수가 **동일하게 증가한다는 점**이다.

<img src = "https://ifh.cc/g/WNg1VB.jpg" width = 300 height = 400>
    
그렇다면 이를 이용해서 알고리즘을 작성해보자!

```
import math
def solution(w,h):
    answer = 0
    n, m = min(w, h), max(w, h)
    gcd = math.gcd(w, h)
    for i in range(1, n//gcd+1):
        answer += math.trunc(-m*i/n+m) + math.trunc(-m*(n-i+1)/n+m)
    return answer*gcd
```
* 여기서 **최대공약수**를 사용하기 위해서 **math.gcd**를 사용했다.

해당 알고리즘은 위 알고리즘보다는 효율적이지만 w, h에 큰 값이 주어질 경우 시간이 오래 걸린다.

<br>

### 🔔best 풀이

반복되는 작은 사각형에서 멀쩡하지 않은 사각형의 개수는 몇 개일까?

아래 사진을 통해서 확인해보자!

<img src = "https://ifh.cc/g/SvzR4T.jpg" width = 400 height = 300>

따라서 작은 사각형 속 멀쩡하지 않은 사각형의 개수는 **'w+h-1'**개이다.
그렇다면 전체 사각형에서의 멀정하지 않은 사각형의 개수는 몇 개일까?

<img src = "https://ifh.cc/g/fBYAoz.jpg" width = 300 height = 400>

사진을 통해서 전체 사각형 속 멀쩡하지 않은 사각형의 개수는 **'w+h-gcd'**인 것을 알 수 있다.

그렇다면 전체 사각형의 개수에서 위 값을 빼게 되면 사각형 속 멀쩡한 사각형의 개수를 구할 수 있게 된다.

```
import math
def solution(w,h):
    return w*h - (w+h-math.gcd(w,h))
```


<br>

## 👩‍💻배운 점
* math 모듈 사용
    * trunc: 버림 / round(수, 자리수): 반올림 / ceil: 올림 / floor: 내림
    * gcd: 최대공약수 / lcm: 최소공배수 

<BR>

🙇‍♀️풀이에 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃 참고
* <https://leedakyeong.tistory.com/entry/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8%EC%8A%A4-%EB%A9%80%EC%A9%A1%ED%95%9C-%EC%82%AC%EA%B0%81%ED%98%95-in-python>
* <https://programmers.co.kr/learn/courses/30/lessons/62048/solution_groups?language=python3&type=all>
* <https://programmers.co.kr/learn/courses/30/lessons/62048>