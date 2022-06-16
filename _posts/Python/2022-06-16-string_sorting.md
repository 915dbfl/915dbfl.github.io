---
title: "[Python] 문자열 정렬"
excerpt: "rjust, ljust, zfill"
categories:
  - Python
tag:
 - python
 - rjust
 - ljust
 - zfill

last_modifeid_at: 2022-06-16
toc: true
toc_sticky: true
search: true
---

<br>

# 👩 오른쪽 정렬: rjust

> **rjust(공백의 수, 공백을 채울 문자)**

```
print("hello".rjust(20, "*")) #***************hello
```

<br>

# 👩 왼쪽 정렬: ljust

> **ljust(공백의 수, 공백을 채울 문자)**

```
print("hello".ljust(20, "*")) #hello***************
```

<br>

# 👩 zfill

> **zfill(공백의 수)**

* 무조건 **오른쪽** 정렬이 된다.
* 공백을 **0으로만** 채울 수 있다.

```
print("hello".zfill(20)) #000000000000000hello
```

<br>


🙇‍♀️부족한 부분이 있다면 말씀해주세요! 감사합니다!


<br>
## 📃 참고
* [https://www.crocus.co.kr/1660]