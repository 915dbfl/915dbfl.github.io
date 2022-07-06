---
title: "[Android] Retrofit 이미지 업로드"
excerpt: "Multipart 서버 통신"
categories:
  - Android
tag:
  - android 
  - kotlin
  - retrofit

last_modified_at: 2022-07-06
toc: true
toc_sticky: true
search: true
---

<br>

👩 이번 시간에는 [retrofit](https://915dbfl.github.io/android/retrofit2/)에 이미지 파일을 업로드 하는 방식에 대해 알아보고자 한다.

바로 그 방식에 대해 알아보기 전! 그 속에서 다뤄지는 multipart에 대해 정리하고 넘어가고자 한다!!

<br>

# 👩 [HTTP](https://ko.wikipedia.org/wiki/HTTP)
 > HTTP는 클라이언트와 서버 사이에 이루어지는 요청/응답(request/response) 프로토콜이다.

 🙄 이미지 파일을 업로드 하는 경우?

  이미지 파일(png, jpg) 자체가 서버로 넘어가는 것이 아니다!! 이미지 파일 또한 문자로 구성되기 때문에 서버로 업로드 될 때 HTTP request body에 **문자 형태**로 전송되게 된다.


😠 그래서! multipart가 도대체 뭔데!

## 🙋‍♀️ HTTP 구조

<img src = "https://ifh.cc/g/GvW2RQ.png" width = 400 height = 400>

위 사진은 HTTP의 구조이다.

데이터를 전송할 때 HTTP **Message Body**에 들어가는 **데이터 타입**을 **HTTP Header에 명시할 수 있다.** 이 때 명시를 위해 사용되는 필드가 **Content-type**이다. 여기 명시되는 데이터 타입 중 하나가 바로 우리가 사용할 **multipart**이다.

<br>

## 🙋‍♀️ Multipart/for-data

👩 서버에 파일을 업로드 할 때, 웹브라우저의 경우 **form**을 통해 파일을 등록한다. [form](https://inpa.tistory.com/entry/HTML-%F0%9F%93%9A-%ED%8F%BCForm-%ED%83%9C%EA%B7%B8-%EC%A0%95%EB%A6%AC)을 구성하는 태그는 name, action, method, enctype 등이 존재하지만 오늘 우리가 알아야 할 부분은 enctype 부분이다!!

> entype: 폼 데이터(form data)가 서버로 제출될 때 데이터가 **인코딩되는 방법을 명시**

* **application/x-www-form-urlencoded(default)**
  * 모든 문자들은 서버로 보내기 전에 인코딩된다.
* **text/plain**
  * 공백 문자(space)는 "+" 기호로 변환되지만, 나머지 문자는 모두 인코딩되지 않는다.
* ** 🔔multipart/form-data🔔**
  * 모든 문자를 인코딩하지 않는다.
  * **파일이나 이미지**를 서버로 전송할 경우 주로 사용된다.

<br>

## 🙋‍♀️ Multipart 등장

HTTP Request Body에 들어가는 데이터는 **한 종류** 타입이 대부분이다. 따라서 Header에 테이터 타입을 명시하는 Contet-type에도 **하나의 타입**만을 명시할 수 있다. 

🙄 예를 들어 text일 경우 text/plain, jpg면 image/jpeg!

<br>

그렇다면 만약 **<u>사진과 사진의 제목을 함께</u>** 서버에 보내는 상황을 생각해보자! 이때는 Content-type을 text/plain으로 해야 할까, image/jpeg로 해야 할까?

🙄 이러한 상황을 위해서 등장한 것이 **Multipart**이다! **하나의 Body**에 대해 **여러 종류의 데이터**를 구분할 수 있도록 말이다!

* Multipart 요청을 보낼 때 HTTP 메소드는 **POST**를 사용한다.
* **Content-Type**은 `multipart/form-data`를 사용한다.

<br>

👩 그렇다면 본격적으로 이미지를 서버에 업로드하는 과정을 알아보자!!

<br>

# 👩 Retrofit Multipart 통신

## 🙋‍♀️ PostMan으로 이미지 업로드

<img src = "https://ifh.cc/g/7AKBk8.png" width = 600 height = 400>

* 우선 서버에 **데이터를 전송**하기 위해 **POST**를 사용한다.
* 데이터가 들어가는 Body를 **form-data type**로 선택한다.
* text / file 중 **file**을 선택한 후, value에 이미지를 업로드 한다.

<br>

## 🙋‍♀️ 코드 상 이미지 업로드 진행 (핵심)

### ☝️ **POST 요청을 구성**

```kotlin
    @POST("/image")
    @Multipart
    fun postImage(
        @Part file: MultipartBody.Part?): Call<ImageData>
```
🙄 원래는 @Body 어노테이션을 사용해 데이터를 한 방에 넘겨주었다면 **Multipart**에서는 **각 요소마다 @Part 어노테이션을 사용해 따로 넣어주어야 한다.**

<br>

### ✌️ **API 사용하기**
  * 업로드할 이미지 path를 통해 **File 객체**를 만든다.
  ```kotlin
  val file = File(path)
  ```
  * File 객체를 **RequestBody**로 변환한다.
  ```kotlin
  val requestBody = RequestBody.create("multipart/form-data".toMediaTypeOrNull(), file)
  ```
  * POST 요청시 필요한 **Multipart.Part**로 변환한다.
  ```kotlin
  val multipartBody = MultipartBody.Part.createFormData("file", file.name, requstBody)
  ```
    * createFormData()의 **첫 인자**는 위 PostMan에서의 **key값**을 적어준다!
  * API 사용하기
  ```kotlin
  ImgApiInterface.create().postImage(multipartBody).enqueue(
      object : retrofit2.Callback<ImageData>{
          override fun onResponse(call: Call<ImageData>, response: Response<ImageData>) {
              ...
              }
          }

          override fun onFailure(call: Call<ImageData>, t: Throwable) {
              Log.d("postImage", "error: $t")
          }
       }
   )
  ```


<br>

  🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃참고
* multipart/form-data 참고 자료
  * <https://velog.io/@shin6403/HTTP-multipartform-data-%EB%9E%80#2-multipart--multipartform-data>
  * <https://inpa.tistory.com/entry/HTML-%F0%9F%93%9A-%ED%8F%BCForm-%ED%83%9C%EA%B7%B8-%EC%A0%95%EB%A6%AC>
  * <https://inpa.tistory.com/entry/HTML-%F0%9F%93%9A-%ED%8F%BCForm-%ED%83%9C%EA%B7%B8-%EC%A0%95%EB%A6%AC>
  * <https://velog.io/@jakeseo_me/Multipart-%EC%A0%95%EB%A6%AC>
  * <https://hastein.tistory.com/4>