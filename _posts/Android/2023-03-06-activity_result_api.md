---
title: "[Android] Activity Resulut API"
excerpt: "startActivityForResult deprecated 대안"
categories:
  - Android
tag:
  - Activity Result Api
  - ActivityResultContract
  - startActivityForResult deprecated
  - 런타임 권한 요청
  - onRequestPermissionsResult deprecated
  - registerForActivityResult
  - RequestPermission
  - RequestMultiplePermissions

last_modified_at: 2023-03-06
toc: true
toc_sticky: true
search: true
---

# 🙋‍♀️ 활동에서 결과 가져오기
카메라 앱을 실행해 사진을 촬영하거나, 갤러리를 실행해 사진을 가져올 때 우리는 **그 결과값을 받아와 사용하는 경우**가 대부분이다.

`startActivityForResult`와 `onActivityResult`
아마 카메라, 갤러리 등의 활동에 대한 결과를 받아올 때 위 api를 많이 사용했을 것이다. 하지만 현재 해당 api는 **deprecated**된 상태이다.

이를 대체하는 것이 바로 **`Activity Result API`**이다.


<br>

# 🙋‍♀️ Activity Result API
* androidx activity와 fragment에 도입되었다.
    * 따라서 해당 api 사용을 위해 위 dependency를 추가해줘야 한다.
        ```kotlin
            implementation "androidx.activity:activity-ktx:[최신 버전]"
        ```


* `registerForActivityResult` api를 통해 결과 콜백을 등록한다.
    * `ActivityResultContract`와 `ActivityResultCallback`을 함께 사용해 **다른 활동을 실행하는 데 사용할** `ActivityResultLauncher`를 반환한다.

<br>

## ➡️ ActivityResultContract
결과를 생성하는 데 필요한 **입력 유형과 결과의 유형**을 정의한다. 

사진 촬영, 권한 요청 등과 같은 기본 인텐트 작업의 [기본 계약](https://developer.android.com/reference/androidx/activity/result/contract/ActivityResultContracts)을 제공하며, [자체 맞춤 계약](https://developer.android.com/training/basics/intents/result?hl=ko#custom)을 정의할 수도 있다.

<br>

## ➡️ ActivityResultCallback
**ActivityResultContract로 인한 결과값을 가져오는** onActivityResult 메서드만을 포함한 단일 메서드 인터페이스이다.

```kotlin
val getContent: ActivityResultLauncher = registerForActivityResult(GetContent()) { uri: Uri? ->
    // handle the returned Uri
}
```
-> ActivityResultContract의 GetContent를 통해 갤러리를 실행하고 선택한 사진의 uri를 받아와 처리한다.

<br>

# 🙋‍♀️ registerForActivityResult
단순히 **콜백을 등록하는 역할을 한다.** 따라서 다른 활동을 실행하거나 결과 요청을 시작하지 않는다.

`launch`를 통해 **결과를 생성하는 프로세스가 실행된다.**

```kotlin
val getContent = registerForActivityResult(GetContent()) { uri: Uri? ->
    // Handle the returned Uri
}

override fun onCreate(savedInstanceState: Bundle?) {
    // ...

    val selectButton = findViewById<Button>(R.id.select_button)

    selectButton.setOnClickListener {
        // Pass in the mime type you'd like to allow the user to select
        // as the input
        getContent.launch("image/*") //⭐️⭐️⭐️
    }
}
```

<br>

# 🙋‍♀️ ActivityReulstContract로 권한 요청
ActivityResultContract를 통해 deprecated된 `onRequestPermissionsResult`를 대신할 수도 있다.

* **RequestMultiplePermissions** : 여러 권한 동시에 요청
    * An ActivityResultContract to request permissions
* **RequestPermission** : 단일 권한 요청
    * An ActivityResultContract to request a permissions

바로 위의 두 계약서를 사용하는 것이다!!

## ➡️ RequestPermission 사용 예시

```kotlin
// Register the permissions callback, which handles the user's response to the
// system permissions dialog. Save the return value, an instance of
// ActivityResultLauncher. You can use either a val, as shown in this snippet,
// or a lateinit var in your onAttach() or onCreate() method.
val requestPermissionLauncher =
    registerForActivityResult(RequestPermission()
    ) { isGranted: Boolean ->
        if (isGranted) {
            // Permission is granted. Continue the action or workflow in your
            // app.
        } else {
            // Explain to the user that the feature is unavailable because the
            // feature requires a permission that the user has denied. At the
            // same time, respect the user's decision. Don't link to system
            // settings in an effort to convince the user to change their
            // decision.
        }
    }
```

위의 코드는 공식 문서에 나와있는 RequestPermission 사용 예시이다. 
이 또한 마찬가지로 콜백을 등록만 했을 뿐, 권한 요청을 진행한 것은 아니다.

```kotlin
    requestPermissionLauncher.launch(android.Manifest.permission.CAMERA)
```
권한 요청은 위에서도 설명했듯이 `launch`를 통해 비로소 실행된다.

<br>

## ➡️ RequestMultiplePermissions 사용 예시

```kotlin
private val requestPermissionLauncher =
    registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { permissions ->
        permissions.entries.forEach { permission ->
            when {
                // 권한 승인 여부에 따라 분기 처리
            }
        }
    }
```

사용법이 RequestPermission과 유사한 것을 알 수 있다. 
이 또한 launch를 통해 여러 권한을 한꺼번에 전달함으로써 권한 승인을 진행할 수 있다.

```kotlin
companion object {
    private val REQUIRED_PERMISSIONS = arrayOf(
            Manifest.permission.ACCESS_COARSE_LOCATION,
            Manifest.permission.ACCESS_FINE_LOCATION
    )
}

...

requestPermissionLauncher.launch(REQUIRED_PERMISSIONS)
```

<br>

# 👀 registerForActivityResult 사용 시 주의사항

> registerForActivityResult는 프래그먼트나 활동을 만들기 전에 호출하는  것이 안전하므로 반환되는 ActivityResultLauncher 인스턴스의 멤버 변수를 선언할 때 직접 사용할 수 있다.


# 📑 주요 내용 정리

<img src = "https://drive.google.com/uc?id=1BOTkRkwle3TRMJx29V3280pwXE-PORsj" width = 600 height = 500>

<br>

**🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!**

<br>

## 📃참고
* 공식 문서
  * <https://developer.android.com/training/basics/intents/result?hl=ko>
  * <https://developer.android.com/training/permissions/requesting?hl=ko>
* 그 외
 * <https://cliearl.github.io/posts/android/request-runtime-permission/>