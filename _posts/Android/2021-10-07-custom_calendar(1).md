---
title: "[Android/Calendar] 커스텀 캘린더 구현(1): 이번 달 출력하기"
excerpt: "GridView 사용해 이번 달 출력하기!"
categories:
  - Android
tag:
  - android 
  - Custom Calendar
  - 안드로이드 달력
  - 커스텀 달력
  - android calendar

last_modified_at: 2021-10-10
toc: true
toc_sticky: true
search: true
---

## Intro
👩 안드로이드에서는 기본적인 **CalendarView 컴포넌트**를 제공한다. 해당 요소는 캘린더의 최소한의 기능을 제공하지만 내가 사용하고자하는 방식으로 커스텀할 수는 없다.

<br>

이번 게시글에서는 원하는 스타일을 적용한 커스텀 캘린더 구현 방식에 대해서 알아보려고 한다.

<br>

그럼 시작해보자고~!🏃‍♀️

## 👩 필요한 xml 구성
🙋‍♀️ 필자는 캘린더를 **GridView**를 통해서 구현하고자 한다.

또한, gridview에 **어댑터**를 등록하여 각 날짜를 오늘, 이전 달, 다음 달...에 따라서 다르게 표시하려고 한다.


<br>

그렇다면 캘린더를 구현하기 위해서는 **총 두가지 layout**이 필요하다.
* **전체 캘린더 구조** 구성 xml 파일
* 캘린더 속 **각 날짜**를 나타내는 xml파일


지금부터 우선 캘린더 전체 layout부터 구성해보자!!

### 👩‍💻custom_calendar.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_margin="10dp"
android:padding="10dp"
android:orientation="vertical"
android:layout_width="match_parent"
android:layout_height="match_parent">

<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:gravity="center"
    android:paddingTop="12dp"
    android:paddingBottom="12dp"
    android:paddingLeft="30dp"
    android:paddingRight="30dp">

    <ImageView
        android:id="@+id/btn_nmonth"
        android:layout_width="15dp"
        android:layout_height="15dp"
        android:src="@drawable/n_month"/>

    <TextView
        android:id="@+id/text_cmonth"
        android:textSize="25dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textStyle="bold"
        android:text= "Current"/>


    <ImageView
        android:id="@+id/btn_bmonth"
        android:layout_width="15dp"
        android:layout_height="15dp"
        android:src="@drawable/b_month"/>
</LinearLayout>

<LinearLayout
    android:id="@+id/calendar_header"
    android:layout_width="match_parent"
    android:layout_height="40dp"
    android:orientation="horizontal"
    android:gravity="center_vertical">

    <TextView
        android:textStyle="bold"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:gravity="center_horizontal"
        android:text="SUN"/>
    
    // ... 위 TextView와 같이 월, 화, 수, 목, 금, 토를 구성한다.
</LinearLayout>

<GridView
    android:id="@+id/calendar_days"
    android:numColumns="7"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"/>
</LinearLayout>
```
* **numColumns**?
  * 한 행에 나열할 column의 수로 일주일을 표시할 것이므로 7로 지정!
  * **꼭** 지정해야 하는 GridView 속성 중 하나!

<br>

👩 그렇다면 이제는 각 날짜를 구성할 xml파일을 생성하자!

### 👩‍💻calendar_day.xml
```
<?xml version="1.0" encoding="utf-8"?>
<TextView
    android:layout_margin="5dp"
    android:layout_width="35dp"
    android:layout_height="35dp"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:text="1"
    android:textSize="18dp"
    android:gravity="center"/>

```
* 각 날짜에 따라 해당 요소 스타일을 변경한 후 gridview에 추가할 것이다.

<br>

여기까지 진행했다면 필요한 레이아웃은 모두 생성한 것이다.

이제부터는 본격적으로 달력을 구성해보자!


## 👩 Component class 만들기
위에서 구성한 **custom_calendar.xml**을 위한 **component class**를 만들 것이다.

<br>

🙄 그렇다면 <u>왜 activity나 fragment로 만들지 않는 것일까?</u>

컴포넌트 클래스로 구성할 경우, 코드의 반복을 예방하고 모듈식 설계를 진행할 수 있다. 따라서 하나의 모듈이 하나의 책임을 가지고 처리할 수 있게 된다.

<br>

그 이유도 알았다면 이제! component class를 작성하자!

해당 게시글에서는 코드의 중요 부분별로 나눠 설명을 진행하겠다!

전체 소스코드는 링크된 [깃 레포](https://github.com/915dbfl/youlAndroid)를 참고하길 바란다.


<br>

### 👩‍💻CalendarView
* custom_calendar.xml의 **root**는 **LinearLayout**이므로 해당 클래스는 **LinearLayout을 상속**받는다.

* <u>initControl: CalendarView.kt</u>
  * 클래스 **생성자**에 들어가는 함수로 생성될 때 실행되게 된다.
  * 따라서 사용할 **xml layout의 요소를 로드**하는 부분이다.
    ```
        private fun initControl(context: Context){
            val inflater = context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
            inflater.inflate(R.layout.custom_calendar, this)
            gridView = findViewById(R.id.calendar_days)
            textCmonth = findViewById(R.id.text_cmonth)
        }
    ```
  <br>  

* <u>updateCalendar: CalendarView.kt</u>
  * 캘린더 속 하나의 달의 날짜를 구성하는 방식은 세부적으로 다음과 같은 과정이 필요하다.

    * 해당 달의 **1일이 시작하는 요일**을 알아야 한다.
    * 해당 위치 **이전**에는 **전 달의 날짜**가 역순으로 들어가게 된다.
    * 해당 달 날짜를 모두 표시한 후, 남은 영역에는 **다음 달의 시작 날짜들**이 들어게가 된다.

    ```
    fun updateCalendar(events: HashSet<Date>, inputCalendar: Calendar){
        // 캘린더에 들어갈 날짜를 저장할 변수
        val cells = ArrayList<Date>()

        // 이번 달의 1일이 사작하는 셀 위치를 구한다.
        inputCalendar.set(Calendar.DAY_OF_MONTH, 1)
        val monthBeginningCell = inputCalendar.get(Calendar.DAY_OF_WEEK) -1

        // 이번 달 첫 셀에 들어갈 이전 달 날짜로 캘린더의 day 설정
        inputCalendar.add(Calendar.DAY_OF_MONTH, -monthBeginningCell)

        // 날짜를 1씩 증가하며 값을 cell에 저장한다.
        while(cells.size < 42){
            cells.add(inputCalendar.time)
            inputCalendar.add(Calendar.DAY_OF_MONTH, 1)
        }

        // gridview의 어댑터 설정
        gridView.adapter = CalendarAdapter(
            context, cells, events, inputCalendar.get(Calendar.MONTH)
        )

        // 달력 상단에 해당 월을 표시
        val calInstance = Calendar.getInstance()
        val sdf: SimpleDateFormat = SimpleDateFormat("MMM")
        textCmonth.text = sdf.format(calInstance.time)
    }
    ```

    🙄 방식을 조금 더 세부적으로 살펴보면 다음과 같다.

    * 이 달의 **1일**로 day를 설정한다.

    * **1일일 때 요일**을 1-7 중 하나의 숫자 값으로 가져온다.(a)
      
      * **이번 달 1일**에서 **(a -1)을 빼주면** 이번 달의 시작 날짜에 들어올 **이전 달의 날짜**를 구할 수 있다.

    * Calendar.DAY_OF_MONTH 값을 **1씩 증가**하며 이번 달에 넣을 날짜 값들을 배열에 저장!

      <br>

    🙋‍♀️ <u>여기서 잠깐!!!</u>
    * cell의 최대 크기가 **<u>42</u>**인 이유?
      * gridView의 행이 **가장 많은 경우**는 달의 1일이 **첫 주의 토요일부터 시작하는 경우**이다.
      * 그럴 경우, 달력에는 총 **6행 7열**의 칸이 필요하게 되므로 cell의 최대값은 42가 된다.

    아래 그림을 함께 참고한다면 위 설명이 보다 잘 이해가 될 것이다!

    ![달력계산](https://ifh.cc/g/8Cwml8.jpg)

    <br>

## 👩 Adapter 구현

우리가 지금까지 진행한 부분을 정리해보겠다.

* 캘린더 전체 layout, 날짜 layout 구성
* component class 구성, 해당 달에 들어갈 날짜들 배열에 정리

그렇다면 이제부터 해야 할 작업은 날짜들이 들어있는 배열의 각 날짜를 이전 달, 현재 달, 다음 달, 오늘과 같은 특징에 따라 스타일을 적용해 **하나의 calendar_day 뷰**를 만드는 것이다.

<br>

👩 어댑터에 대해서 조금 더 제사한 내용을 알아보고 싶다면 [안드로이드 어댑터 공식 문서](https://developer.android.com/reference/android/widget/Adapter#constants)를 참고하길 바란다.
  * getView()에 대한 내용도 참고하면 도움이 될 것 것이다.


<br>

그렇다면 adapter 구현 코드를 확인해보자.

```
class CalendarAdapter(context: Context, days: ArrayList<Date>, eventDays: HashSet<Date>, inputMonth: Int): ArrayAdapter<Date>(context, R.layout.custom_calendar, days){
    private val inflater: LayoutInflater = LayoutInflater.from(context)
    // 넘어온 inputMonth 값의 경우, 현재 달이 아닌 다음 달을 나타내므로 -1을 한다.
    private val inputMonth = inputMonth - 1

    // getView를 통해 데이터 셋에서 특정 위치에 있는 데이터를 나타낼 뷰를 가져와 작업한다.
    override fun getView(position: Int, view: View?, parent: ViewGroup): View {

        var view = view
        val calendar = Calendar.getInstance()

        // 해당 position의 데이터 값을 가져온다.
        val date = getItem(position)

        // 가져온 데이터 값으로 캘린더 설정
        calendar.time = date
        val day = calendar.get(Calendar.DATE)
        val month = calendar.get(Calendar.MONTH)
        val year = calendar.get(Calendar.YEAR)

        // 캘린더 값과 비교할 오늘 날짜를 가져온다.
        val today = Date()
        val calToday = Calendar.getInstance()
        calToday.time = today

        // 뷰 설정
        if(view == null){
            view = inflater.inflate(R.layout.calendar_day, parent, false)
        }

        // TextView 기본 설정
        (view as TextView).setTypeface(null, Typeface.NORMAL)
        (view as TextView).setTextColor(Color.BLACK)

        // 이번 달이 아닌 경우 회색으로 표시
        if(month != inputMonth || year != calToday.get(Calendar.YEAR)){
            view.setTextColor(Color.parseColor("#C4C4C4"))
        }
        // 오늘인 경우, 파란색으로 표시
        else if(day == calToday.get(Calendar.DATE)){
            view.setTextColor(Color.parseColor("#56a6a9"))
        }

        // 날짜 설정
        view.text = calendar.get(Calendar.DATE).toString()

        return view
    }

}
```
* 우리가 다룰 data set은 **배열**이므로 adapter는 **ArrayAdapter**를 상속한다.

나머지 코드의 경우, 색을 설정하고, 글자를 지정하는 등 단순히 스타일링하는 부분으로 설명을 생략하도록 한다.

## 👩 activity에 설정

이제 마지막이다!!

👩 위에서 열심히 만든 CalendarView가 **화면에 출력될 수 있도록** activity에 설정하기만 하면 끝!

### 👩‍💻activity.xml
* 캘린더 뷰를 출력할 액티비티의 layout으로 다음 요소를 작성해준다.
```
    <com.cookandroid.dev.CalendarView
        android:id="@+id/calendar_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
```

### 👩‍💻Activity.kt
* **activity.xml**의 **CalendarView를 로딩**해 **updateCalendar**를 호출한다.
```
        ...

        val cv = binding.calendarView as CalendarView
        val calInstance = Calendar.getInstance()
        cv.updateCalendar(events, calInstance)

        ...
```

<br>

여기까지 진행한다면 다음과 같은 결과를 얻을 수 있을 것이다!!

![캘린더결과](https://ifh.cc/g/0v8Ur3.png)

<br>

히히..좀 뿌듯하지 않은가?
~~(필자는 좀 많이 뿌듯하다!)~~

그렇다면 일!단! 이번 달 출력은 성공이다!🙌

이번 게시글은 여기서

끝! ~(˘▾˘~)


## ✍ GIR REPO
* [https://github.com/915dbfl/youlAndroid](https://github.com/915dbfl/youlAndroid)

## 📃참고

* https://developer.android.com/reference/android/view/LayoutInflater
* https://developer.android.com/reference/android/view/LayoutInflater
* https://www.toptal.com/android/android-customization-how-to-build-a-ui-component-that-does-what-you-want