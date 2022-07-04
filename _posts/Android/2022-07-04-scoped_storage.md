---
title: "[Android] Scoped Storage"
excerpt: "안드로이드 Q 이후의 저장소"
categories:
  - Android
tag:
  - android 
  - kotlin

last_modified_at: 2022-07-04
toc: true
toc_sticky: true
search: true
---


👩 이번 시간에는 안드로이드 저장소에 대해 자세히 살펴보고, 안드로이드 빌드 버전 Q(10, API 29) 이상부터 달라진 접근 방식에 대해 알아보자!

<BR>

## 👩 내부 / 외부 저장소

안드로이드 저장소는 크게 내부 / 외부 저장소로 나뉘며, 각 저장소의 특징은 다음과 같다.

* **내부 저장소**
  * 저장소 **해당 앱**에서만 접근, 저장, 삭제가 가능하다.
  * 앱이 삭제될 경우, 저장소 내 파일이 **모두** 삭제된다.
  * 사용자, 다른 앱에서의 접근이 **불가**하다.

* **외부 저장소**
  * 파일을 생성한 앱이 아닐 경우, 접근하기 위해서는 **권한**이 필요하다.
  * 앱이 삭제될 경우, 외부 저장소의 앱 전용 공간의 파일만이 모두 삭제되고, **공용 공간의 파일들은 삭제되지 않는다.**
  * 외부 저장소의 **공유 공간**에 파일을 저장함으로써 다른 앱과 공유가 가능해진다.


<BR>

## 👩 Scoped Storage

| 안드로이드 Q 이전 저장소(Legacy Storage)

<img src = "https://ifh.cc/g/xGRRbm.png" width = 600 height = 400>

🙄 안드로이드 Q 이전 외부 저장소는 **하나의 큰 공용 공간**으로 운영되면서 다음과 같은 문제점이 존재했다.

* READ_EXTERNAL_STORAGE / WRITE_EXTERNAL_STORAGE 권한만 있으면 **모든 파일에 접근/수정이 가능하다.**
* 각각의 파일이 어느 앱에 속하는지 알 수 없었기에 관리가 어렵다.
* 앱을 삭제해도 파일이 저장공간을 계속 차지하고 있는다.

<br>

안드로이드 Q 이상부터는 **Scoped Storage**가 적용되었다!

| 안드로이드 Q 이후 저장소(Scoped Storage)

<img src = "https://ifh.cc/g/4bn3qY.png" width = 600 height = 400>

* 내부 저장소
  * 내부 저장소의 개별 앱 공간은 이전과 동일하다.

* 외부 저장소
  * 외부 저장소의 개별 앱 공간이 **샌드박스 형식**으로 보호되는 공간을 할당받는다.
    * **샌드박스?** 프로그램이 **보호된 영역**에서 동작해 부정하게 조작되는 것을 막는 보안 형태
  * public files, 공용 공간이 **사진 및 동영상, 음악, 다운로드로 나뉘었다.**

<br>

👩 위와 같이 Scoped Storage(범위 지정 저장소) 구조로 바뀌면서 외부 저장소의 **파일 접근 방식**에도 변화가 발생했다. 지금부터 그 방식에 대해서 하나씩 살펴보자!!

<br>

## ☝️외부 저장소 접근: 개별 앱 공간
* 내부 저장와 비슷해 다른 앱에서는 접근할 수 **없다.**
* 앱 자신의 개별 앱 공간에 접근할 때는 별도의 권한 요청이 필요 **없다.**
* **getExternalFilesDir()/getExternalCacheDir()**을 통해 접근이 가능하다.
* 앱이 삭제될 경우 그 속 파일들이 함께 삭제된다.

<br>

## ✌️외부 저장소 접근: 공용 공간 속 사진/동영상/음악
* 자신이 생성한 파일에 대해서는 불필요한 권한 요청을 하지 않아도 된다.
* 자신이 생성한 파일이 아닐 경우, 권한을 요청해야 한다.
  * **MediaStore 사용**
    * READ_MEDIA_IMAGES
    * READ_MEDIA_VIDEO
    * READ_MEDIA_AUDIO
  * **MediaStore 혹은 다른 방법을 사용할 경우**
    * READ_EXTERNAL_STORAGE (단, 필요한 권한만을 요청하는 것을 권고)
* **MediaStorage API**를 사용해 접근한다.
  * MediaStorage API: 사용자 파일을 다른 앱에서도 사용할 수 있도록 설계된 API
* 앱을 삭제하더라도 함께 제거되지 않는다.


<br>

## 👌외부 저장소 접근: 다운로드
* 권한은 필요없으며, **Storage Access Framework**와 **시스템 파일 선택기**를 통해 사용자가 선택한 특정 파일에 접근이 가능해진다.
* 앱을 삭제하더라도 제거되지 않는다.

### 🙄 Storage Access Framework
  * **다운로드 공간**에 접근하기 위해 사용된다.
  * **시스템 유아이**를 통해서 클라이언트 앱과 프로바이더가 상호작용한다.
    * ACTION_OPEN_DOCUMENT / ACTION_CREATE_DOCUMENT 인텐트를 실행해 상호작용이 가능해진다.

<br>

🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃참고
* <https://developer.android.com/training/data-storage/shared/media?hl=ko>
* <https://developer.android.com/training/data-storage/app-specific?hl=ko>
* <https://youngest-programming.tistory.com/386>
* <https://developer.android.com/training/data-storage/use-cases?hl=ko>
* <https://velog.io/@bonimddal2/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-QAPI-29%EC%9D%B4%EC%83%81%EC%97%90%EC%84%9C-%EC%9D%B4%EB%AF%B8%EC%A7%80-%ED%8C%8C%EC%9D%BC-%EB%8B%A4%EB%A3%A8%EA%B8%B0>
* <https://bacassf.tistory.com/31>