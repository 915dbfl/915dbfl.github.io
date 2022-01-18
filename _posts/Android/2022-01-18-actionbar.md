---
title: "[Android] ActionBar에 액션 설정하기"
excerpt: "showAsAction 설정"
categories:
  - Android
tag:
  - android 
  - RecyclerView
  - android recyclerview
  - kotlin
  - ActionBar
  - menu

last_modified_at: 2022-01-18
toc: true
toc_sticky: true
search: true
---

## Intro

액션바는 안드로이드 화면 가장 상단에 존재하는 바를 나타낸다. 이번 시간에는 액션바에 액션들을 추가하고, 특정 액션을 고정시키는 작업을 진행해볼 것이다.

<br>

## 🙋‍♀️액션 추가하기(메뉴)

![액션들](https://ifh.cc/g/pXg13a.png)

* 우선 여기서 말하는 **액션**은 메뉴 속 각 **항목**들을 의미한다.
* 각각의 액션은 menu 속 하나의 item으로 존재하게 된다.
  * res -> menu에 메뉴에 해당하는 메뉴 xml파일을 작성한다.

```
# main_menu.xml

<menu xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <item
        android:id="@+id/item1"
        android:title="메뉴1"/>
    <item
        android:id="@+id/item2"
        android:title="메뉴2" />
    <item
        android:id="@+id/item3"
        android:title="메뉴3"/>
    <item
        android:id="@+id/item4"
        android:title="메뉴4"/>
</menu>
```

* 여기까지 진행했다면 우리는 해당 액션들을 액션바에 적용해야 한다.

<br>

## 🙋‍♀️액션바에 액션들 적용하기: onCreateOptionsMenu



```
# MainActivity.kt

override fun onCreateOptionsMenu(menu: Menu?): Boolean {
  //액션들 설정
  menuInflater.inflate(R.menu.main_menu, menu)
  return true
    }
```

* 해당 함수의 반환값을 **true**로 할 경우, 액션바 오른쪽에 메뉴들을 선택할 수 있는 버튼이 생기게 된다.

👩 그렇다면 특정 액션을 액션바에 고정시켜 보자.

<br>

## 🙋‍♀️showAsAction

* 액션을 **고정**시키기위해서는 액션들에 **showAsAction** 값을 설정해야 한다.

* 액션바는 공간이 제한적이므로 한정된 수의 액션들만 고정시킬 수 있다.

* 따라서 showAsAction에 고정시키는 여부를 달리할 수 있다.

  |showAsAction| 기능 |
  |-------------|------|
  |always| 액션바에 **항상** 고정|
  |ifRoom| **공간이 있다면**   액션바에 고정|
  |never|액션바에 표시하지 않음    (basic)|
  |withText|액션바에 표시할 때,   공간이 있을 경우 타이틀 값과 같이 표시함|
  |collapseActionView|액션바에   **커스텀 액션 뷰**를 보여줌|


<br>

👩 위 사진 속 메뉴버튼 옆에 존재하는 액션은 아래의 코드를 통해서 액션바에 고정할 수 있다.

```
# main_menu.xml

    <item
        android:id="@+id/item1"
        android:icon="@drawable/calendar_b_month"
        android:title="메뉴1"
        app:showAsAction="always|collapseActionView"
        app:actionViewClass="androidx.appcompat.widget.SearchView" />
```

<br>

## 🙋액션 이벤트 처리하기: onOptionsItemSelected
```
# MainActivity.kt

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
      // 선택된 action의 id에 따라서 이벤트를 처리함
        when(item.itemId){
            R.id.item1 ->{
              ...
            }
            ...
        }
        return super.onOptionsItemSelected(item)
    }
```

<br>

## 📃참고
* [https://recipes4dev.tistory.com/141](https://recipes4dev.tistory.com/141)
* 인프런 윤재성의 Kotlin 기반 안드로이드 앱 개발 Part3 수강