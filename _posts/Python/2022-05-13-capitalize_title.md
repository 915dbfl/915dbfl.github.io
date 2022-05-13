---
title: "[Python] capitalize vs title"
excerpt: "대문자로 바꾸는 다양한 방법"
categories:
  - Python
tag:
 - python
 - functools.cmp_to_key
 - capitalize
 - title

last_modifeid_at: 2022-05-13
toc: true
toc_sticky: true
search: true
---

<br>

# 👩 capitalize()
* **문자열의 첫글자**는 대문자로, 나머지는 소문자로 변환하는 함수이다.

```
str1 = "hello, world!!"
str2 = "HELLO, WORLD!!"

print(str1.capitalize()) #"Hello, world!"
print(str2.capitalize()) #"Hello, world!"
```

🙄 그렇다면 문자열에 숫자가 들어가 있으면 어떻게 될까?

```
str1 = "3hello, world!!"
str2 = "hello, 3world!!"

print(str1.capitalize()) #"3hello, world!!"
print(str2.capitalize()) #"Hello, 3world!!"
```
* 첫 글자에 숫자가 오더라도 **첫 글자를 제외한 나머지는 소문자로 변환한다**.

<br>

# 👩 title()
* **띄어쓰기 기준**으로 문자열 내 **모든 단어**의 첫 글자는 대문자로, 나머지는 소문자로 변환하는 함수이다.

```
str1 = "hello, world!!"
str2 = "HELLO, WORLD!!"

print(str1.title()) #"Hello, World!"
print(str2.title()) #"Hello, World!"
```

🙄 그렇다면 title()의 경우도 숫자가 포함되어 있는 문자열을 살펴보자!

```
str1 = "3hello, world!!"
str2 = "Hello, 3WORLD!!"

print(str1.title()) #"3Hello, World!!"
print(str2.title()) #"Hello, 3World!"
```
* title의 경우 무조건 첫 글자를 대문자로 변환하는 것이 아닌 **숫자가 아닌 첫 알파벳**을 **대문자로 변환**하는 것을 확인할 수 있다.

<br>

----

이번 게시글은 여기서 마치도록 하겠다!

🙇‍♀️부족한 부분이 있다면 말씀해주세요! 감사합니다!


<br>
## 📃 참고
* [https://zetawiki.com/wiki/%ED%95%A8%EC%88%98_capitalize()]