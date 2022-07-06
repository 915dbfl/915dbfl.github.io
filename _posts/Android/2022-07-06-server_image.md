---
title: "[Android] 서버에 저장된 이미지 가져오기"
excerpt: "다운로드 vs 불러오기"
categories:
  - Android
tag:
  - android 
  - kotlin
  - Glide

last_modified_at: 2022-07-06
toc: true
toc_sticky: true
search: true
---

<br>

👩 지난 시간에는 서버에 이미지를 저장하는 방식에 대해서 알아보았다! 이번 시간에는 서버에 저장한 이미지를 사용하는 방식에 대해서 알아볼 것이다.

서버 이미지를 사용하는 방식은 크게 두 가지가 존재한다.

> **다운 + 비트맵 그리기 vs url로 이미지 불러오기**

만약 다룰 이미지의 개수가 많다면 이미지마다 다운받아 비트맵으로 그리는 방식을 사용한다면 오랜 시간이 걸리게 된다. 따라서 많은 이미지를 처리할 때는 url로 이미지를 불러오는 방식을 사용하는 것이 좋다!

🙋‍♀️ 그렇다면 지금부터 두 가지 방식에 대해 자세히 살펴보자!

<br>


## 👩 이미지 다운로드 + 비트맵 그리기

해당 방식의 과정을 자세히 살펴보면 다음과 같다!
* 소켓 통신으로 이미지를 **다운**받는다.
* 다운받은 이미지를 바탕으로 **비트맵을 생성**한다.
* imageView에 비트맵을 셋팅한다.

☝️ 우선 소켓 통신을 하기 위해서는 AndroidManifest에 **인터넷 권한**을 설정해야 한다.

```kotlin
<uses-permission android:name="android.permission.INTERNET"/>
```

✌️ 이제 코드를 통해 방법을 자세히 살펴보자!

```kotlin
    try{
        val url = URL(userData.user.imagePath)
        val conn = url.openConnection()
        conn.connect() # 연결
        val bis = BufferedInputStream(conn.getInputStream())
        val bm = BitmapFactory.decodeStream(bis)
        bis.close() # 종료
        userImg.setImageBitmap(bm)
    }catch(e: IOException){
        Log.e("dialogSetImage", e.toString())
    }
```
* URL객체를 생성해 openConnection으로 **URLConnection/HttpURLConnection**오브젝트를 얻어온다.
  * **URLConnection**: URL에 대한 API 제공
  * **HttpURLConnection**: URLConnection의 서브 클래스, HTTP 고유 기능에 대한 추가 지원을 제공한다.
  * 해당 오브젝트를 활용해 <u>실제 네트워크 연결 전</u> 클라이언트와 서버 간의 다양한 옵션을 설정할 수 있다.


  -> 🙄 더 나아가 자세한 내용을 알고 싶다면 다음 게시글을 참고하길 바란다!
    * [URLConnection / HttpURLConnection?](https://blueyikim.tistory.com/2199)

<br>

* 데이터를 읽기 위해 inputStream을 가져온다. (getInputStream)
  * getInputStream에서 발생할 수 있는 예외
    * IOException: inputStream을 생성하는 동안 I/O 오류 발생
    * SocketTimeoutException: 읽기 제한 시간이 만료되는 경우
    * UnkownServiceException: 프로토콜에서 입력을 지원하지 않는 경우
  * [🙄BufferedInputStream?](https://oper6210.tistory.com/12)

<br>

## 👩 url로 이미지 바로 불러오기 : Glide

웹 주소로 되어 있는 이미지를 바로 불러오기 위해 사용되는 여러 라이브러리가 존재한다.

해당 게스글에서 필자는 대부분의 사람들이 사용하느 **Glide**를 사용해보고자 한다!!

> [Glide: https://github.com/bumptech/glide](https://github.com/bumptech/glide)


<br>

### 🙋‍♀️ Glide란?
* 구글에서 공개한 이미지 라이브러리이다.
* 웹 이미지를 로드할 때 필요한 고려사항을 미리 구현해 사용자가 쉽게 이용할 수 있도록 한다.

<br>

### 🙋‍♀️ 사용법

  ☝️ 라이브러리 다운
  ```kotlin
  dependencies {
    implementation 'com.github.bumptech.glide:glide:4.13.2'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.13.2'
  }
  ```

  ✌️ 사용하기
  ```kotlin
  Glide.with(context).load(url).error(source).into(target)
  ```

  😮그렇다! 이 한 문장으로 웹 상 이미지를 불러와 사용할 수 있게 된다.

  * **into(target)**를 이미지를 보여줄 이미지 뷰를 지정한다.
  * **error(source)**를 통해서 원본 이미지를 로드할 수 없을 경우 보여질 이미지를 설정할 수 있다.

  <br>

  👩 위에서 살펴본 기본 이미지를 로딩하는 방식에서 나아가 placeholder(), donAnimate() 등을 호출해 Glide를 다양하게 사용할 수 있다!! 
  
  자세한 사항은 다음 게시글을 참고하길 바란다!!
  * [Glide 사용법?](http://dktfrmaster.blogspot.com/2016/09/glide.html)
  

<br>

  🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃참고
* url 이미지 다운 관련 참고
  * <https://binsolb.tistory.com/4>
  * <https://blueyikim.tistory.com/2199>
  * <https://oper6210.tistory.com/12>
* Glide 관련 참고
  * <http://dktfrmaster.blogspot.com/2016/09/glide.html>
  * <https://wonpaper.tistory.com/m/207>
