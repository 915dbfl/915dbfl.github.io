---
title: "[Android] ActionView"
excerpt: "collapseActionView를 통한 ActionView 설정"
categories:
  - Android
tag:
  - android 
  - RecyclerView
  - android recyclerview
  - kotlin
  - ActionView

last_modified_at: 2022-01-18
toc: true
toc_sticky: true
search: true
---

## Intro

액션바에 적용하는 아이템들에게 showAsAction을 설정하여 액션바에 직접 보이도록 설정할 수 있다. 

이번 시간에는 **showAsAction 값을 collapseActionView로 설정함**으로써 ActionView를 설정해보고자 한다.

<br>

![사진](https://ifh.cc/g/7vKtKA.png)

* ActionView의 경우 위 사진과 같이 검색기능을 위해 많이 사용된다고 한다.

👩 따라서 이번 시간에는 검색기능을 한 번 구현해보고자 한다.

<br>

## 🙋‍♀️showAsAction = "collapseActionView"

```
# main_menu.xml

    <item
        android:id="@+id/item1"
        android:icon="@drawable/calendar_b_month"
        android:title="메뉴1"
        app:showAsAction="always|collapseActionView"
        app:actionViewClass="androidx.appcompat.widget.SearchView" />

```
* 값을 설정하다보면 android 위젯에서 제공하는 것과 androidx에서 제공하는 것이 있다.
  * androidx를 선택해야 하위버전에도 잘 적용이 된다.
  * androidx를 선택했을 때, 잘 작동이 안된다면 그때 android에서 제공하는 것을 선택하자.

## 🙋‍♀️searchView의 펼쳐진 여부에 따른 리스너 설정

### 🙋‍♀️searchView 객체 가져오기

```
#MainActivity.kt

    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.main_menu, menu)

        //searchview를 가지고 있는 메뉴 아이템 객체를 추출
        val item1 = menu?.findItem(R.id.item1)
        //searchview 객체를 추출
        val search1 = item1?.actionView as SearchView
        //안내문구를 설정한다.
        search1.queryHint = "검색어 입력"

      return true
    }
```

### 🙋‍♀️리스너 객체 생성
```
# MainActivity.kt
# 위 코드 안에 작성

        val listener1 = object: MenuItem.OnActionExpandListener{
            //접혔을 때 호출

            override fun onMenuItemActionCollapse(item: MenuItem?): Boolean {
             ...   
            }

            //펼쳐졌을 때 호출
            override fun onMenuItemActionExpand(item: MenuItem?): Boolean {
              ...
            }
        }

```

* 반환되는 값은 크게 상관이 없다.
* searchView를 가지고 있는 메뉴 아이템이 각 함수에 item에 해당된다.

### 🙋‍♀️item에 리스너 적용

```
#MainActivity.kt
#위 코드와 이어짐

item1.setOnActionExpandListener(listener1)
```

## 🙋‍♀️SearchView에 대한 리스너

* 값이 입력될 때마다 호출되는 함수와 검색이 완료된 후 호출되는 함수가 존재한다.

```
# onCreateOptionsMenu 안에 작성

val listener2 = object: SearchView.OnQueryTextListener{
            //searchView에 값이 입력될 때마다 호출되는 함수
            override fun onQueryTextChange(newText: String?): Boolean {
                textView1.text = "입력 중"
                textView2.text = "입력: $newText"

                return true
            }
            //키보드의 돋보기 버튼을 눌렀을 때 호출되는 함수
            //즉, 입력이 완료되었을 때 호출되는 함수
            override fun onQueryTextSubmit(query: String?): Boolean {
                textView1.text = "문자열 입력 완료"
                textView2.text = "입력 완료: $query"

                //입력완료 후, 키보드가 사라지게 함
                search1.clearFocus()
                return true
            }
        }

        // 리스너 적용
        search1.setOnQueryTextListener(listener2)
        
        return true
}
```

<br>

## 📃참고
* 인프런 윤재성의 Kotlin 기반 안드로이드 앱 개발 Part3 수강