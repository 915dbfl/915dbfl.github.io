---
title: "[Android] DialogFragment 사용하기"
excerpt: "BaseDialog 구성 후 Dialog 구현하기"
categories:
  - Android
tag:
  - DialogFragment

last_modified_at: 2023-02-17
toc: true
toc_sticky: true
search: true
---

# 🙋‍♀️ DialogFragment?

기존 Dialog와 달리 이름에서 알 수 있듯이 fragment의 한 종류이다. 즉, dialog를 보여주는 데 활용되는 fragment이다.

그렇다면 기존 Dialog와 차이점은 무엇일까?
* DialogFragment를 활용하면 **Fragment의 생명주기**를 활용할 수 있다.
  * 기존 Dialog를 활용할 때 Activity가 파괴되더라도 dialog가 존재하여 leaked window crash, IllegalStateException 문제가 발생했다.
  * 따라서 **수명 주기 이벤트를 올바르게 처리할 수 있게 된다.**

그렇다면 본격적으로 DialogFragment를 활용해보자!!

<br>

# 🙋‍♀️ BaseDialogFragment

우선 dialog 같은 경우, 다양한 곳에서 자주 사용될 것 같아 base dialog를 만들어 두었다.

```kotlin
abstract class BaseDialog<T: ViewBinding> (private val layoutResId: Int) : DialogFragment() {

    private var _binding: T? = null
    protected val binding get() = _binding!!
```
base fragment이기 때문에 **abstract**으로 구성하고 **viewbinding에 사용될 값을 generic으로 받는다.**

또한, 해당 fragment와 연결될 **layout을 매개변수로 받는다.**

```
    override fun onDestroy() {
        _binding = null
        super.onDestroy()
    }
```

fragment의 경우, 화면에 나타나지 않을 때 binding값을 반환해줘야 하기 때문에 binding 변수는 크게 _binding, binding으로 구성한다.

```kotlin
    abstract fun getViewBinding(): T
```

```kotlin
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        dialog?.apply {
            window?.setBackgroundDrawable(ColorDrawable(Color.TRANSPARENT)) // dialog가 화면에 떴을 때 배경을 투명하게!
        }

        _binding = getViewBinding()
        return binding.root
    }
```

generic에서는 binding을 바로 할 수 없기 때문에 해당 DialogFragmentr가 사용될 때 getViewBinding을 구현해 _binding 값이 정의될 수 있도록 하였다.

```
    override fun onResume() {
        super.onResume()

        // 디바이스의 size 얻기
        val display = resources.displayMetrics
        // 비율로 너비 지정
        val window: Window = dialog?.window ?: return
        val params: WindowManager.LayoutParams = window.attributes
        params.width = (display.widthPixels * 0.8).toInt()
        //적용
        dialog?.window!!.attributes = params
    }
```

dialog의 layout은 기본적으로 match_parent의 너비를 가지게 된다. 따라서 onResume에서 이에 대한 처리가 필요하다!

따라서 디스플레이의 사이즈를 얻고 그에 따른 비율로 너비를 지정해주었다.

이렇게 해서 BaseDialog를 만들었다! 그렇다면 이를 활용해 실제 사용할 dialog를 구현해보자!!

<br>

# 🙋‍♀️ BaseDialog 활용하기

```
class HomePermissionDialog : BaseDialog<FragmentHomePermissionDialogBinding>(R.layout.fragment_home_permission_dialog) {
```
위에서 정의한 **BaseDialogFragment**를 활용해 사용할 Dialog를 구성한다.

여기서 generic으로 viewbindg 값을 지정하고, 연결될 layout을 매개변수로 넘겨준다.

```
override fun getViewBinding() = FragmentHomePermissionDialogBinding.inflate(layoutInflater)
```
generic으로 바로 사용하지 못했던 viewbinding을 지정하기 위해 getViewBinding을 구현해 정의해준다.


```
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        super.onCreateView(inflater, container, savedInstanceState)
        binding.dialogCameraBtn.setOnClickListener {
            dismiss() // dialog 종료
        }

        return binding.root
    }
```
**onCreateView**를 override해 버튼의 클릭 리스너 등의 view 작업을 진행한다. 

여기서 dismiss는 dialog를 종료시키는 코드이다. 이때 사용할 수 있는 코드가 cancel과 dismiss, 두가지가 존재한다. 이 둘의 차이점은 [해당 블로그](http://sjava.net/2011/12/android-dialog의-cancel과-dismiss의-차이점/)를 통해 자세히 알 수 있으니 참고 바란다!

여기까지 진행한다면 모든 작업이 마무리된다!! 

~~(주절주절)사실 나는 base를 생성해본 적이 한 번도 없다...ㅎㅎㅎ 지금 개발을 진행하면서 한 레포를 참고하고 있는 데! 생각보다 훨씬 많은 것을 배우고 있는 것 같다!! 오랜만에 블로그를 작성하니 역시 기록이 학습을 되새기는 방식으로는 짱이라는 것을 
다시 한 번 느낀다. 앞으로 달려보자! 아자아자!!(주절주절)~~


<br>

**🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!**

<br>

## 📃참고
* 공식 문서
  * <https://developer.android.com/guide/topics/ui/dialogs?hl=ko>
  * <https://developer.android.com/reference/android/app/DialogFragment#Lifecycle>
* 그 외
  * <https://bb-library.tistory.com/258>
  * <http://sjava.net/2011/12/android-dialog의-cancel과-dismiss의-차이점/>
  * <https://stackoverflow.com/questions/66702149/basefragment-with-viewbinding>