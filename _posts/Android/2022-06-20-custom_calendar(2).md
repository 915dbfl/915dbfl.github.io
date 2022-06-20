---
title: "[Android] 커스텀 캘린더 구현(2): RecyclerView"
excerpt: "RecycleView를 통해 커스텀 캘린더 구현하기!"
categories:
  - Android
tag:
  - android 
  - Custom Calendar
  - 안드로이드 달력
  - 커스텀 달력
  - android calendar
  - recyclerview custom calendar

last_modified_at: 2022-06-20
toc: true
toc_sticky: true
search: true
---


👩 이전 시간에 [**GridView**를 통해서 커스텀 캘린더](https://915dbfl.github.io/android/custom_calendar(1)/)를 제작해 사용해보았다. 하지만 이를 사용하면서 내가 겪었던 불편한 점은 <u>특정 아이템 뷰들만 업데이트할 수 없었다는 점이다.</u>

그래서 이번 시간에는 저번에 구현한 GridView 커스텀 캘린더를 **RecyclerView**로 바꿔 구현해 봄으로써 추가적으로 **특정 뷰만을 업데이트하는 방식**에 대해서도 학습해 볼 것이다.

특정 달의 달력을 구성하고, 이전/이번/다음 달에 따라 글자색을 달리 하는 등과 같은 세부적인 내용은 GridView를 통해 구현한 커스텀 캘린더에서 사용한 방식을 그대로 사용하므로 자세한 내용은 다음 게시글을 참고하길 바란다!
* [👩GridView 커스텀 캘린더](https://915dbfl.github.io/android/custom_calendar(1)/)


이번 포스팅에서는 RecyclerView로 바꿔 구현하는 과정에서 달라진 점을 중심으로 정리해보고자 한다!

<br>

# 🙋‍♀️xml 구성하기

커스텀 캘린더를 구현하기 위해 필요한 xml은 다음과 같이 두 가지이다.
* **전체 캘린더 구조** 구성 xml
* 캘린더 속 **각 날짜**를 나타내는 xml 
  * 👌 GridView 때와 동일하므로 생략!
  
<br>

## 👩 전체 캘린더 구조 구성 xml: custom_calendar.xml

```
        🔔GridView 대신 **RecyclerView** 사용!🔔
        <androidx.recyclerview.widget.RecyclerView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="10dp"
            android:layout_marginRight="10dp"
            android:layout_marginLeft="10dp"
            android:id="@+id/calendar_days"
            android:orientation="vertical"
            🔔sapnCount를 통해서 열의 크기를 지정한다.🔔
            app:spanCount="7"
            🔔layoutManager도 GridLayouManager로 지정해준다.🔔
            app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"/>
```
* 여기서 레이아웃 매니저는 목록의 개별 요소를 정렬하는 역할을 하게 된다.

<br>
----

# 🙋‍♀️ 컴포넌트 클래스: 캘린더뷰 클래스
  * GridView 때와 동일하므로 생략!!

<br>
----

# 🙋‍♀️ 캘린더 뷰의 어댑터🔔
* 이전에는 ArrayAdapter를 상속받아 구현했다면 이제는 **RecyclerView.Adapter**를 상속해 구현해야 한다.

* **RecyclerView와 그 Adapter**에 대해서 자세히 알고 싶다면 다음 공식 문서를 참고하길 바란다!!
  * [👩 RecyclerView로 동적 목록 만들기](https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ko)

<br>

## 👩 ViewHolder
ViewHolder는 RecyclerView의 목록에 있는 **개별 항목의 레이아웃**을 포함하는 View의 래퍼이다.

따라서 목록을 구성하는 아이템 레이아웃에서 사용할 **요소**가 있다면 이곳에 모두 **정의**해야 한다.

필자가 구현하는 커스텀 달력의 경우, 목록을 구성하는 아이템에서 사용할 요소는 날짜를 나타내는 textView만 존재하기 때문에 이를 ViewHodler에 정의해주었다.

```
    inner class ViewHolder(private val view: View): RecyclerView.ViewHolder(view){
      val textView: TextView //레이아웃 속 요소

        init {
            textView = view.findViewById(R.id.textView)//바인딩
            view.setOnClickListener {
                ...
                //view 리스너
            }
        }
    }
```

* 위 코드처럼 뷰 홀더로 넘겨지는 **view에 클릭 리스너를 달아** 클릭 시 동작을 정의할 수 있다.

<br>

## 👩 onCreateViewHolder
* **ViewHodler를 새로 만들때마다** 호출되는 메소드이다.
* **ViewHolder**와 **그와 연결된 View**를 생성하고 초기화한다.
  * 단, 특정 데이터와는 아직 **연결되지 않은** 틀과 같은 형태이다!

```
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.calendar_day, parent, false)
        return ViewHolder(view)
    }
```
<br>

## 👩 onBindViewHolder
* 해당 단계에서 **특정 데이터**와 **ViewHodler**가 **연결**되게 된다.
* **position에 위치하는 데이터**를 가져와 ViewHodler에서 사용하게 된다!

```
    override fun onBindViewHolder(viewHolder: ViewHolder, position: Int) {
        // 캘린더 날짜의 글자 색을 달리하는 작업이 이곳에서 이루어진다!
        viewHolder.textView.text = dataSet[position]
    }
```
* 해당 단계를 통해서 캘린더 속 날짜들이 구성되게 된다.
* 따라서 해당 메소드에 날짜의 색을 결정하는 코드가 들어가게 된다.

<br>

## 👩 getItemCount()
* RecyclerView에서 총 몇 개의 데이터가 있는지 확인할 때 사용하며, 필수적으로 overide 해야 한다.

```
override fun getItemCount() = dataSet.size
```
* 여기서 dataSet은 어댑터로 넘겨진 데이터셋이다.


<br>
----

# 🙋‍♀️ 커스텀 뷰 사용: activity에 설정

GridView로 구현했을 때와 동일하게 적용하면 된다!

<br>

----
# ➕ 특정 아이템 업데이트: notifyItemChanged(인덱스)

여기까지 진행했다면 RecyclerView를 통해 커스텀 캘린더가 완성되었을 것이다!
그렇다면 이제 하나의 아이템만을 업데이트하는 방식을 살펴보자!!

코드는 정말 간단하다!
```
notifyItemChanged(아이템 인덱스)
```
<br>

이를 이용해서 특정 아이템을 클릭했을 때 기존에 클릭된 아이템과 새로 클릭된 아이템이 모두 업데이트 되도록 코드를 작성해보자!

과정은 이렇다!

* 우선 아이템 뷰에 **position**을 **태그**로 저장할 것이다.
* 뷰가 클릭될 때마다 뷰에서 태그를 꺼내 값을 저장한다.
* 뷰가 클릭될 때 기존의 태그값과 새롭게 클릭된 뷰의 태그값을 사용해 두 아이템을 업데이트한다.

<br>

그렇다면 코드를 통해서 살펴보자!

```
class CalendarAdapter(private val context: Context, private val days: ArrayList<Date>, private val inputMonth: Int, private val calendarViewModel: CalendarViewModel, private val type: Int): RecyclerView.Adapter<CalendarAdapter.ViewHolder>(){

  ...
    var pos = 0 //🙄현재 클릭되어 있는 뷰의 position을 저장함
  ...

    inner class ViewHolder(private val view: View): RecyclerView.ViewHolder(view){
      val textView: TextView
        init {
            textView = view.findViewById(R.id.textView)
            view.setOnClickListener {
                val newIndex = it.tag as Int //🙄뷰의 태그를 통해 새롭게 클릭된 뷰의 position을 꺼내온다.
                val oldIndex = pos //🙄기존에 클릭되어 있던 뷰의 position
                this@CalendarAdapter.pos = newIndex //🙄pos값 업데이트

                //🙄 positon을 통해 두 아이템 업데이트
                notifyItemChanged(newIndex)
                notifyItemChanged(oldIndex)
            }
        }
        ...

        fun bind(position: Int){
          view.tag = position
          textView.text = dataSet[position]
        }

    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
      holder.bind(position)
    }
    ...
}
```

이렇게 RecyclerView를 통해서 커스텀 캘린더를 구현하는 방식에 대해서 알아보았다!

어댑터의 형태를 제외하고는 거의 비슷한 과정으로 커스텀 캘린더를 구현할 수 있다. 하지만 뷰의 업데이트에 있어 GridView보다는 RecyclerView를 사용할 때 훨씬 효과적으로 동작을 구현할 수 있다는 것을 알 수 있었다.

<br>

## 📃참고
* <https://furang-note.tistory.com/29>
* <https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ko>
* <https://shwoghk14.blogspot.com/2020/10/android-custom-calendar-with.html>
8 <https://mashup-android.vercel.app/mashup-10th/miinjung/CustomCalendar/>
