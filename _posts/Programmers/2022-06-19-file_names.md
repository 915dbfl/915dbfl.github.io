---
title: "[Programmers/ level2] [3차] 파일명 정렬"
excerpt: "정규표현식 그룹핑 적용!"
categories:
  - Programmers
tag:
 - python
 - programmers
 - level2
 - 파일명 정렬 3차
 - regex
 - regex group()

last_modifeid_at: 2022-06-19
toc: true
toc_sticky: true
search: true
---

<br>



# 👩문제

세 차례의 코딩 테스트와 두 차례의 면접이라는 기나긴 블라인드 공채를 무사히 통과해 카카오에 입사한 무지는 파일 저장소 서버 관리를 맡게 되었다.

저장소 서버에는 프로그램의 과거 버전을 모두 담고 있어, 이름 순으로 정렬된 파일 목록은 보기가 불편했다. 파일을 이름 순으로 정렬하면 나중에 만들어진 ver-10.zip이 ver-9.zip보다 먼저 표시되기 때문이다.

버전 번호 외에도 숫자가 포함된 파일 목록은 여러 면에서 관리하기 불편했다. 예컨대 파일 목록이 ["img12.png", "img10.png", "img2.png", "img1.png"]일 경우, 일반적인 정렬은 ["img1.png", "img10.png", "img12.png", "img2.png"] 순이 되지만, 숫자 순으로 정렬된 ["img1.png", "img2.png", "img10.png", img12.png"] 순이 훨씬 자연스럽다.

무지는 단순한 문자 코드 순이 아닌, 파일명에 포함된 숫자를 반영한 정렬 기능을 저장소 관리 프로그램에 구현하기로 했다.

소스 파일 저장소에 저장된 파일명은 100 글자 이내로, 영문 대소문자, 숫자, 공백(" "), 마침표("."), 빼기 부호("-")만으로 이루어져 있다. 파일명은 영문자로 시작하며, 숫자를 하나 이상 포함하고 있다.

<br>

파일명은 크게 HEAD, NUMBER, TAIL의 세 부분으로 구성된다.

* HEAD는 숫자가 아닌 문자로 이루어져 있으며, 최소한 한 글자 이상이다.
* NUMBER는 한 글자에서 최대 다섯 글자 사이의 연속된 숫자로 이루어져 있으며, 앞쪽에 0이 올 수 있다. 0부터 99999 사이의 숫자로, 00000이나 0101 등도 가능하다.
* TAIL은 그 나머지 부분으로, 여기에는 숫자가 다시 나타날 수도 있으며, 아무 글자도 없을 수 있다.

<br>

|파일명|head|number|tail|
|------|-----|----|-----|
|foo9.txt|foo|9|.txt|
|foo010bar020.zip|foo|010|bar020.zip|
|F-15|F-|15|(빈 문자열)|


파일명을 세 부분으로 나눈 후, 다음 기준에 따라 파일명을 정렬한다.

* 파일명은 우선 HEAD 부분을 기준으로 사전 순으로 정렬한다. 이때, 문자열 비교 시 대소문자 구분을 하지 않는다. MUZI와 muzi, MuZi는 정렬 시에 같은 순서로 취급된다.
* 파일명의 HEAD 부분이 대소문자 차이 외에는 같을 경우, NUMBER의 숫자 순으로 정렬한다. 9 < 10 < 0011 < 012 < 13 < 014 순으로 정렬된다. 숫자 앞의 0은 무시되며, 012와 12는 정렬 시에 같은 같은 값으로 처리된다.
* 두 파일의 HEAD 부분과, NUMBER의 숫자도 같을 경우, 원래 입력에 주어진 순서를 유지한다. MUZI01.zip과 muzi1.png가 입력으로 들어오면, 정렬 후에도 입력 시 주어진 두 파일의 순서가 바뀌어서는 안 된다.

무지를 도와 파일명 정렬 프로그램을 구현하라.

<br>

## 👩입력형식

입력으로 배열 files가 주어진다.

* files는 1000 개 이하의 파일명을 포함하는 문자열 배열이다.
* 각 파일명은 100 글자 이하 길이로, 영문 대소문자, 숫자, 공백(" "), 마침표("."), 빼기 부호("-")만으로 이루어져 있다. 파일명은 영문자로 시작하며, 숫자를 하나 이상 포함하고 있다.
* 중복된 파일명은 없으나, 대소문자나 숫자 앞부분의 0 차이가 있는 경우는 함께 주어질 수 있다. (muzi1.txt, MUZI1.txt, muzi001.txt, muzi1.TXT는 함께 입력으로 주어질 수 있다.)

<br>

## 👩입출력 예

* 입력: ["img12.png", "img10.png", "img02.png", "img1.png", "IMG01.GIF", "img2.JPG"]
    * 출력: ["img1.png", "IMG01.GIF", "img02.png", "img2.JPG", "img10.png", "img12.png"]

* 입력: ["F-5 Freedom Fighter", "B-50 Superfortress", "A-10 Thunderbolt II", "F-14 Tomcat"]
    * 출력: ["A-10 Thunderbolt II", "B-50 Superfortress", "F-5 Freedom Fighter", "F-14 Tomcat"]

<br>

## 🙋‍♀️접근 방법

header, number, tail에 따라서 정렬이 되는 것이기 때문에 우선 파일명에서 각 부분의 값들을 구해야 한다.

여기서** header, number까지만 비교해도 정렬이 완료**가 되므로 tail은 무시해도 상관없다!

문자열에서 해당되는 부분을 뽑아내기 위해 사용할 수 있는 것이 바로 **정규식**이다.

<br>

## 🙋‍♀️코드 

코드를 통해서 그 방식을 살펴보자!

```
import re

def solution(files):
    lst = []
    for i, file in enumerate(files):
        header = re.split('[0-9]+', file)[0].upper()
        number = int(re.findall('[0-9]+', file)[0])
        lst.append((header, number, file))
    lst.sort(key = lambda x: (x[0], x[1]))
    return list(i[2] for i in lst)
```
* 오직 숫자로만 이루어진 number 부분을 기준으로 **split**을 진행하면 첫 번째 값에서 header를 구할 수 있게 된다.

* 또한, number의 값은 숫자로만 이루어진 첫부분이 해당되게 된다.

* [🙄split? findall?이 뭐지?](https://915dbfl.github.io/python/regex-(2)/)

<br>

# 🔔best 풀이

* 정규표현식 그룹핑**을 통해 원하는 부분만을 쏙쏙 뽑아낼 수 있다!!
    * [🙄 정규표현식 grouping](https://915dbfl.github.io/python/regex_group/)

코드가 비교적 간단하니 코드를 통해서 자세히 살펴보자!

```
import re

def solution(files):

    def key_function(file):
        header, number = re.match(r"([a-z-. ]+)([0-9]{,5})", file).groups()
        return [header, int(number)]

    return sorted(files, key = lambda file: key_function(file.lower()))
```

* **"([a-z-. ]+)"**를 통해서 **문자만**으로 이루어진 header를 얻게 된다.
* **"([0-9]{,5})"**를 통해서 **최대 다섯 글자의 연속된 숫자**로 이루어진 number를 얻게 된다.
* **[header, number]**를 **sorted 메소드의 key**로 적용해 header로 우선 정렬 후, header가 같을 때 number로 정렬함으로써 정답을 얻게 된다.

<br>

## 👩‍💻배운 점
* **()**을 사용해 **정규표현식 그룹핑**이 가능하다. 이를 통해서 각 그룹에 해당하는 문자열을 **각각** 얻을 수 있다.
* **groups()**를 통해서 매치된 전체 값들을 **튜플 형태**로 얻을 수 있다.

<BR>

🙇‍♀️풀이에 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃 참고
* <https://blockdmask.tistory.com/413>
* <https://programmers.co.kr/learn/courses/30/lessons/68645/solution_groups?language=python3>
* <https://programmers.co.kr/learn/courses/30/lessons/68645>