---
title: "[Android] DrawerLayout"
excerpt: "Navigation Drawer Activity ์ฌ์ฉ๋ฒ!!"
categories:
  - Android
tag:
  - android 
  - kotlin
  - DrawerLayout

last_modified_at: 2022-01-27
toc: true
toc_sticky: true
search: true
---

## Intro

๐โโ๏ธ ์ด๋ฒ ํฌ์คํ์์๋ ์๋๋ก์ด๋์์ ๊ธฐ๋ณธ์ ์ผ๋ก ์ ๊ณตํ๋ **Navigation Drawer Activity**์ฌ์ฉ๋ฒ์ ๋ํด์ ์ ๋ฆฌํด๋ณด๊ณ ์ ํ๋ค.

ํด๋น ๋ฐฉ์์ ์๊ธฐ ์ , ํ์๋ drawer ์์ฑ์ ํ์ํ header layout๊ณผ menu ๋ฑ์ ์ง์  ๊ตฌํํ๋ฉฐ ๋ณต์กํ ๋ฐฉ๋ฒ์ผ๋ก drawer๋ฅผ ์ ์ํ์๋ค.

๊ทธ๋ฐ๋ฐ ์ด๋ ๊ฒ ์ฌ์ด ๋ฐฉ๋ฒ์ด ์์๋ค๋...๐ฅบ
์ง๊ธ๋ถํฐ ๊ทธ ์ฌ์ฉ๋ฒ์ ์์ธํ ์ ๋ฆฌํด๋ณด๊ณ ์ ํ๋ค!!

<br>

## Navigation Drawer Activity๐โโ๏ธ

๐ฉ ์ฐ์  ์๋ก์ด ์กํฐ๋นํฐ๋ฅผ ์์ฑํ  ๋, Navigation Drawer Activity๋ฅผ ์ ํํ๋ค.

<img src= "https://ifh.cc/g/dsho8g.png" width = 500 height= 700> 


## ๊ตฌ์ฑ๐โโ๏ธ

๐ฉ ์ด์ ๋ถํฐ ์๋์ ์ผ๋ก ์์ฑ๋ ํ์ผ๋ค์ ์ฐ์์๋ฅผ ํ์ธํด๋ณด์!

* **activity_main.xml**
  * ํด๋น ํ์ผ์ ํ์ธํด๋ณด๋ฉด ์๋ ๊ตฌ์ฑ์ ํ์ธํ  ์ ์๋ค.

  <img src = "https://ifh.cc/g/Q8mA1B.png" width = 500 height = 600>

  * **include**: ๋ค๋ฅธ layout์ ํฌํจ์ํค๋ ์์๋ก **ํ๋๊ทธ๋จผํธ๊ฐ ๊ต์ฒด**๋๋ ํ๋ฉด์ ๋ํ๋ธ๋ค.
    * ์ต์ข์ ์ผ๋ก content_main.xml์ fragment๊ฐ controller ์ญํ ์ ํ๊ฒ ๋๋ค.

    ### - controller?๐ฉ

      * contorller๋ fragment๋ค์ ๊ด๋ฆฌํ๊ณ , ๊ฐ fragment๋ค์ ๊ต์ฒด๋ฅผ ์ฒ๋ฆฌํ๋ค.

  <br>

  * **navigationview**: ์ข์ธก์ ๋ํ๋๋ ๋ฉ๋ด์ ํด๋น๋๋ค.
    * **headerLayout**: ์ข์ธก ๋ฉ๋ด ์ ๋ถ๋ถ์ ๊ตฌ์ฑํ๋ layout์ ์ง์ ํ๋ค.
    * **menu**: ์ข์ธก ๋ฉ๋ด๋ฅผ ๊ตฌ์ฑํ  menu ํ์ผ์ ์ง์ ํ๋ค.

<br>

----

* **navigation -> mobile_navigation.xml**

  * controller๊ฐ ๊ด๋ฆฌํ  **fragment๋ค ๋ฑ๋ก**ํ๋ xml

  * ๋ฉ๋ด ์์ดํ ํด๋ฆญ์, **์์ดํ์ ์ด๋ฆ๊ณผ ๋์ผํ fragment**๊ฐ ํ๋ฉด์ ๋ํ๋๊ฒ ๋๋ค. (ํต์ฌ!!๐)


  ์ฆ, ์๋์ action_main_drawer.xml ์ ์์ดํ์ ์ด๋ฆ๊ณผ ๋์ผํ ์ด๋ฆ์ ๊ฐ์ง mobile_navigation.xml์ fragment๊ฐ ํ๋ฉด์ ์ถ๋ ฅ๋๊ฒ ๋๋ ๊ฒ์ด๋ค.

  <img src = "https://ifh.cc/g/105jxS.png" width = 500 height = 600>

  <img src = "https://ifh.cc/g/VanGPp.png" width = 500 height = 600>

  
## ํต์ฌ ์ ๋ฆฌ๐

๐ฉ ๊ฒฐ๋ก ์ ์ผ๋ก drawer์ ์ฌ์ฉํ๊ธฐ์ํด ๊ผญ ํ์ํ ๋ถ๋ถ๋ง ์ ๋ฆฌํ์๋ฉด ๋ค์๊ณผ ๊ฐ๋ค.

* drawer์ **์๋ก์ด ๋ฉ๋ด**๋ฅผ ์ถ๊ฐํ๊ณ ์ ํ๋ค๋ฉด menu์ <u>action_main_drawer.xml</u>์ **item์ ์ถ๊ฐ**ํด์ค๋ค.

* **๋์ผํ ์ด๋ฆ์ ๊ฐ์ง fragment**๋ฅผ <u>mobile_navigation.xml</u>์ ์ถ๊ฐํด์ค๋ค.

* drawer์ **ํค๋ ๋ชจ์**์ ๋ฐ๊พธ๊ณ  ์ถ๋ค๋ฉด <u>activity_main.xml</u> ์  NavigationView์ **hearderLayout**์ ์ํ๋ layout์ ๊ตฌ์ฑํด ์ง์ ํ๋ค.

<br>

## ๐์ฐธ๊ณ 
* ์ธํ๋ฐ ์ค์ฌ์ฑ์ Kotlin ๊ธฐ๋ฐ ์๋๋ก์ด๋ ์ฑ ๊ฐ๋ฐ Part3 ์๊ฐ