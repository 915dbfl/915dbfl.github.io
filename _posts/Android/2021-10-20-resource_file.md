---
title: "[Android] Shape drawable"
excerpt: "shape drawable resource를 추가해 요소에 특정 shape 적용하기"
categories:
  - Android
tag:
  - android 
  - shape drawable
  - android shape drawable
  - kotlin

last_modified_at: 2021-10-20
toc: true
toc_sticky: true
search: true
---

## Intro

이번 시간에는 **Shape Drawable**을 통해서 적용하고자 하는 모양의 파일을 구성해 이를 요소에 적용해 원하는 모양으로 구성하고자 한다.

<br>
👩 기본적으로 요소들의 모양을 수정할 때는 xml 파일 속 다양한 속성들을 적용해 수정한다. 
해당 방식으로 여러 요소에 동일한 모양을 적용하고 싶다면 모든 요소에 각각 속성을 적용해줘야 한다.
<br>

이러한 수고를 **drawable resource**를 추가함으로써 해결할 수 있다!

이번 시간에는 그 중 **<u>Shapedrawable</U>**에 대해서 알아보고자 한다!

<br>

## 👩 Drawable Resource
* ["https://developer.android.com/guide/topics/resources/drawable-resource?hl=ko"](https://developer.android.com/guide/topics/resources/drawable-resource?hl=ko)

우선 **Drawable Resource**가 무엇인지를 살펴보자. 

안드로이드 홈페이지에서도 알 수 있듯이 드로어블 리소스는 단어 해석 그대로 **화면에 그릴 수 있는 것**을 의미한다. 모양, 색 등과 같이 화면에 표시될 수 있는 것을 정의하는 파일이다.

<br>

## 👩 ShapeDrawable
* 위에서 말한 상황처럼 여러 요소에 **모서리가 둥근 모양**을 적용하고 싶다고 가정해보자!

  * 이럴 때 사용하는 것이 바로 **ShapeDrawable**이다.

👩 우선 resource file을 res/drawable 폴더 안에 새로 추가한다.

<br>

### 🙋‍♀️ resource file 추가!
  res/drawable에 **drawable resoucre file**을 추가한다.

  그렇다면 다음과 같은 형태의 파일이 생성될 것이다.

  ```
  <?xml version="1.0" encoding="utf-8"?>
  <selector xmlns:android="http://schemas.android.com/apk/res/android">

  </selector>
  ```
  👩 우선 shape drawable을 사용하기 위해서는 **shape**가 **루트 요소**여야 한다.

  ```
  <?xml version="1.0" encoding="utf-8"?>
  <Shape xmlns:android="http://schemas.android.com/apk/res/android">

  </shape>
  ```

<br>

### 🙋‍♀️ ShapeDrawable 요소 살펴보기
홈페이지에 제시된 ShapeDrawable 구문을 통해 적용할 수 있는 요소들을 살펴보자!

```
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape=["rectangle" | "oval" | "line" | "ring"] >
    <corners
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
    <gradient
        android:angle="integer"
        android:centerX="float"
        android:centerY="float"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear" | "radial" | "sweep"]
        android:useLevel=["true" | "false"] />
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer" />
    <size
        android:width="integer"
        android:height="integer" />
    <solid
        android:color="color" />
    <stroke
        android:width="integer"
        android:color="color"
        android:dashWidth="integer"
        android:dashGap="integer" />
</shape>
```

* **android: shpae**
  * 기본적인 **모양 유형**을 정의한다.
  * rectangle, oval, line, ring 중 하나의 모양으로 정의할 수 있다.

* **corners**
  * shape의 **둥근 모서리**를 생성한다.
  * android: shape의 값이 **rectangle**일 경우에는 적용된다.

    ```
      <corners
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
    ```

  * **radius**를 통해 모든 모서리에 값을 한 번에 지정할 수 있다.

* **gradient**
  * 그라데이션 색상을 지정한다.
  ```
      <gradient
        android:angle="integer"
        android:centerX="float"
        android:centerY="float"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear" | "radial" | "sweep"]
        android:useLevel=["true" | "false"] />
  ```
  * 각 요소에 대한 자세한 설명은 [공식 홈페이지](https://developer.android.com/guide/topics/resources/drawable-resource?hl=ko)를 참고하길 바란다.

* **padding**
  * 뷰 요소에 적용할 패딩을 지정한다.
  * left, top, right, bottom을 각각 지정할 수 있다.

* **size**
  * shape의 크기를 지정할 수 있다.
  * weight와 width를 지정한다.

* **solid**
  * shape를 채울 단색을 지정한다.

* **storke**
  * shape의 테두리를 지정한다.
  * width로 선의 두께를 지정할 수 있다.
  * datshWidth와 dashWidth는 함께 사용해야 한다.

<br>

## 🙋‍♀️ 둥근 모서리 적용
그렇다면 **둥근모서리의 모양**을 적용할 수 있는 drawable resource file의 코드를 한 번 살펴보자!!

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:padding="10dp"
    android:shape="rectangle" >
    <solid android:color="#FFF" />
    <corners
        android:bottomLeftRadius="15dp"
        android:bottomRightRadius="15dp"
        android:topLeftRadius="15dp"
        android:topRightRadius="15dp" />
</shape>
```

매우 간단해 아마 쉽게 각 속성들이 무엇을 의미하는지 파악할 수 있을 것이다.

이렇게 작성한 drawable resource를 적용할 때는 다음과 같이 **요소의 배경 속성**으로 적용해주면 된다.

```
android:background="@drawable/layout_rectangle"
```

<br>

👩 아래 사진을 살펴본다면 둥근 모서리가 적용된 것을 확인할 수 있다!
![결과](https://ifh.cc/g/FxPwg7.png)


이번 게시글은 여기서 마치도록 한다!

이만! ~(˘▾˘~)


## 📃참고
* https://developer.android.com/guide/topics/resources/drawable-resource?hl=ko
* https://ju-hy.tistory.com/26
