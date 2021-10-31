---
title: "[Android] RecyclerView 사용하기(2)"
excerpt: "RecyclerView 적용해 목록 구현하기"
categories:
  - Android
tag:
  - android 
  - RecyclerView
  - android recyclerview
  - kotlin

last_modified_at: 2021-10-31
toc: true
toc_sticky: true
search: true
---

## Intro

[이전 게시글](http://127.0.0.1:4000/android/recyclerview(1)/)을 통해서 recyclerview를 사용했을 때 좋은 점과 어떠한 방식으로 recyclerview가 적용되는지 살펴보았다.

그렇다면 이제는 본격적으로 recyclerview를 이용해서 목록을 구성해보자!!

<br>

## 🙋‍♀️ RecyclerView 요소 추가하기
* recyclerview.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <androidx.recyclerview.widget.RecyclerView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/recycler_view"/>
</LinearLayout>
```

👩 recyclerview를 표시하고자 하는 화면에 RecycleListView 요소를 추가한다.

<br>

## 🙋‍♀️ RecyclerView item 레이아웃 구성하기
* recyclerview_todo.xml

* RecyclerView 속 각 데이터 를 구성할 뷰 레이아웃이다.

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="horizontal"
    android:background="#EEEBEB"
    android:layout_margin="10dp"
    android:padding="10dp"
    android:gravity="center_vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <CheckBox
        android:layout_width="30dp"
        android:layout_height="20dp"/>
    <TextView
        android:id="@+id/textview"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:text="사과 먹기"/>
</LinearLayout>
```

<br>

## 🙋‍♀️ Adapter 구성하기
* adapter는 공식 홈페이지에서 제공하는 틀을 원하는 방식대로 적용하면 된다.

아래 코드는 [공식 홈페이지](https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ko)에서 제공하는 adapter를 수정한 것이다.

```
class RecyclerViewAdapter (private val dataSet: ArrayList<String>): RecyclerView.Adapter<RecyclerViewAdapter.ViewHolder>(){
    //provide a reference to the type of views that you are using
    //(custom ViewHolder)

    class ViewHolder(view: View): RecyclerView.ViewHolder(view){
        val textView: TextView

        init{
            //define click listener for the viewholder's view
            textView= view.findViewById(R.id.textview)
        }
    }

    //create new views (inboked by the layout manager)
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        //create a new view, which defines the UI fo the list item
        val view = LayoutInflater.from(parent.context).inflate(R.layout.recyclerview_todo, parent, false)

        return ViewHolder(view)
    }

    //replace the contents of a view (invoked by the layout manager)
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        //get element from your dataset at this position and replace the
        //contents of the view with that element
        holder.textView.text = dataSet[position]
    }

    override fun getItemCount() = dataSet.size
}

```

## 🙋‍♀️ adapter, layoutmanager 적용하기

* RecyclerViewActivity.kt

* 화면 속 recyclerview에 **어댑터**와 **레이아웃 매니저**를 적용해준다.

```
class RecyclerViewActivity : AppCompatActivity() {
    private lateinit var binding: RecyclerviewBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = RecyclerviewBinding.inflate(layoutInflater)
        val view = binding.root
        setContentView(view)
        setTitle("RecyclerView")

        //dataset 구성
        val dataSet = ArrayList<String>()
        dataSet.add("사과 먹기")
        dataSet.add("김치 볶음밥 먹기")
        dataSet.add("떡볶이 먹기")
        dataSet.add("마라탕 먹기")
        dataSet.add("감귤 먹기")

        //adapter와 layoutmanager 적용
        val adapter = RecyclerViewAdapter(dataSet)
        binding.recyclerView.adapter = adapter
        binding.recyclerView.layoutManager = LinearLayoutManager(this)
    }
}
```

👩 그렇다면 그 결과를 함께 살펴보자.

![결과](https://ifh.cc/g/QcuEON.png)

* 넣어준 데이터 셋의 요소들이 아이템 뷰가 적용돼 recyclerview에 하나씩 추가된 것을 확인할 수 있다.

그렇다면 이번 게시글은 이만 마무리하겠다.

끝~ ~(˘▾˘~)

<br>

## ✍ GIT REPO
* [https://github.com/915dbfl/youlAndroid](https://github.com/915dbfl/youlAndroid)

<br>

## 📃참고
* https://recipes4dev.tistory.com/154
* https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ko
