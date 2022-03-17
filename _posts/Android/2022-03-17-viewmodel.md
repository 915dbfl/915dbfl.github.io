---
title: "[Android] ViewModel"
excerpt: "UI 컨트롤러 로직에서 뷰 데이터 소유권 분리하는 법!"
categories:
  - Android
tag:
  - android 
  - kotlin
  - viewmodel

last_modified_at: 2022-03-17
toc: true
toc_sticky: true
search: true
---

## 👩ViewModel?
[공식문서](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko)에서의 **ViewModel** 정의는 다음과 같다. 


> ViewModel 클래스는 수명 주기를 고려하여 UI 관련 데이터를 저장하고관리하도록 설계되었습니다.
ViewModel 클래스를 사용하면 화면 회전과 같이 구성을 변경할 때도 데이터를유지할 수 있습니다.


우리는 기존 화면 구성이 변경될 경우, 데이터 유지를 다음과 같이 진행하였다.
아마 모두에게 익숙한 코드일 것이다!!

  ```
  override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState) # 요 친구!
          ...
  }
  ```

* 액티비티가 사라지기 전 유지하고자 하는 데이터를 **saveInstanceState**을 통해 onCreate()에 념겨줌으로써 데이터가 유지된다.
* 하지만 넘길 수 있는 데이터의 양, 형태의 제한이 존재하고 UI 컨트롤러(Activity, Fragment 등)의 할 일이 늘어나 많은 시간이 걸린다는 단점이 존재한다.

<br>

🙄 UI 컨트롤러가 데이터에 관여하지도 않고, saveInstanceState를 사용하지도 않는 방식! 

이것이 바로 **ViewModel**인 것이다!

<br>

## 👩ViewModel의 생명주기
 
그렇다면 ViewModel이 어떻게 위와 같은 문제를 해결할 수 있는 것일까?
그것은 ViewModel의 생명주기와 관련이 있다.

<img src = "https://ifh.cc/g/7aCvXl.png" width = 500 height = 700>

🙄 여기서 핵심은! Activity의 onDestroy()가 호출되어도 소멸되지 않는다.
즉, **액티비티가 재생성이 되는 경우에도 소멸되지 않는다는 것이다!**

finish 메소드가 호출거나 사용자가 직접 백버튼을 눌러 **액티비티가 종료되었을 때** ViewModel이 비로소 소멸되게 된다.

<br>

## 🙋‍♀️ViewModel 사용하기

그렇다면 본격적으로 ViewModel 사용법에 대해서 하나씩 알아보자.

공식문서의 경우, **ViewModel**과 **LiveData**를 함께 사용하는 예제를 보여준다.
  * ViewModel과 LiveData를 함께 사용할 경우, 최대의 효용성을 가지기 때문이다.
  * 여기서는 ViewModel 사용법을 집중적으로 살펴보고자 한다.

### ☝ ViewModel 생성하기

🙄 화면에 점수가 표시되고, 두 버튼을 통해서 점수를 증가/감소시킬 수 있는 예제를 만들어 보고자 한다.

```
class MainViewModel: ViewModel() {
    var score: Int = 0
}
```
* ViewModel()을 상속받아야 한다.
* ViewModel에는 **유지되어야 하는 UI와 관련된 데이터**인 **score**가 선어되어야 한다.

<br>

### ✌ ViewModel 사용: ViewModelProvider
🙄 보통 클래스를 사용하기 위해서는 생성자를 사용하지만! ViewModel의 경우, **ViewModelProvider** 객체를 사용한다.

```
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var mainViewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        //get viewmodel instance⭐⭐⭐
        mainViewModel = ViewModelProvider(this).get(MainViewModel::class.java)

        binding.button.setOnClickListener {
            increase()
        }

        binding.button2.setOnClickListener {
            decrease()
        }


    }

    fun increase(){
        mainViewModel.score += 1 # 데이터를 가져와 사용
        binding.textView.text = mainViewModel.score.toString()
    }

    fun decrease(){
        mainViewModel.score -= 1 # 데이터를 가져와 사용
        binding.textView.text = mainViewModel.score.toString()
    }
}
```

* ViewModelProvider를 통해 ViewModel의 instance를 얻은 후로는 일반 <u>클래스를 사용하듯</u> 사용할 데이터를 가져와 읽고, 쓰면 된다.
* ViewModelProvider(**<u>owner</u>**).get(**<u>modelclass</u>**)⭐⭐⭐
  * owner: ViewModelStore을 소유하고 있는 객체

<br>

## 👩ViewModel 사용 시 주의사항
> **activity, fragment, context에 대한 참조**는 저장하면 안된다.

🙄 이는 ViewModel의 생명주기를 생각한다면 그 이유를 간단히 이해할 수 있다.

ViewModel은 Activity보다 **긴** 생명주기를 가지고 있다. 따라서 Activity의 참조를 가지고 있을 경우, Activity가 **재생성**되어도 <u>종료된 Activity의 참조를 가지고 있게 된다.</u> 그리고 이는 ViewModel이 사라질 때까지 계속 존재하게 된다. 

따라서 이는 **Memory leak 현상**을 발생시킨다.

<br>

## 👩마무리

🙄 이번 시간에는 ViewModel에 대해서 알아보았다. 우연히 알게된 개념인데
LiveData와 함께 사용해서 조금 더 효율적으로 데이터 관리를 할 수 있다는 사실이 너무 흥미로웠다! 이걸 지금이라도 알아서 너무너무너무! 다행이다...! 다음시간에는 ViewModel과 짝꿍인 LiveData에 대해서 알아보고자 한다!!!

## 📃참고
* [https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko)
* [https://todaycode.tistory.com/33](https://todaycode.tistory.com/33)
* [https://jeongmin.github.io/2020/05/04/android/architecture-components/viewmodel/](https://jeongmin.github.io/2020/05/04/android/architecture-components/viewmodel/)
