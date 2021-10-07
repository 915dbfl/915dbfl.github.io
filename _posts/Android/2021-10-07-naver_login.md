---
title: "[Android/Login] 네이버 아이디로 로그인(1)"
excerpt: "네이버 오픈 API 활용하기"
categories:
  - Android
tag:
  - android 
  - naver login api
  - kotlin
  - 네이버 연동 로그인
  - 네이버 아이디 로그인
  - 네아로

last_modified_at: 2021-10-07
toc: true
toc_sticky: ture
search: true
---

## Intro
내 앱에서 네이버 아이디로 로그인을 진행할 수 있도록 **네이버 연동 로그인**을 구현해보고자 한다.
<br>

🏃‍♀️ 안드로이드용 네이버 아이디 로그인 구현 방식에 대해서는 [네아로 안드로이드](https://developers.naver.com/docs/login/api/api.md)
에서 보다 자세히 확인할 수 있으며, 해당 게시글도 위 사이트를 참조해 작성한다.
<br>

그렇다면 시작해보자!!



## 🙋‍♀️요구사항
* 가장 먼저 네이버 아이디로 로그인 라이브러리를 사용하려면 다음과 같은 환경이 필요로 된다.

![요구사항](https://ifh.cc/g/jEypE2.png)


## 🙋‍♀️오픈 API 이용 신청
* 네이버 아이디로 로그인 API를 사용하고자 한다면 **애플리케이션을 먼저 등록**해야 한다.


* 등록을 통해 얻게 되는 **client id**와 **client secret** 값이 후에 **로그인 오픈 API**를 사용하기 위해 필요하게 된다.


<br>
등록과정은 간단해 빠르게 넘어가고자 한다!


<br>

![등록](https://ifh.cc/g/ynEC72.png)

* **애플리케이션 이름**: 네이버 아이디로 로그인할 때 사용자에게 표시되는 이름에 해당


* **사용 API**: 네아로를 통해 가져올 정보들에 해당


* 로그인 오픈 API 서비스 환경
  * **다운로드 URL**: 앱이 **스토어**에 올라가있을 경우, 해당 URL을 작성하면 된다. 단, 스토어에 올라가지 않았다면 특정 **사이트 주소**(ex. 개발 페이지)를 작성하면 된다.


  * **안드로이드 앱 패키지 이름**: 말그대로 **앱의 Manifest.xml에서 확인할 수 있는 패키지 이름**을 작성하면 된다. 
  ![패키지](https://ifh.cc/g/PAxcdp.png)



## 🙋개발환경 설정
* 우선 android용 네아로 라이브러리를 다운받아야 한다.

![라이브러리](https://ifh.cc/g/2o1YTc.png)
* 해당 페이지 깃 허브로 들어가 zip 형태로 다운로드를 진행한다.


* 네아로 라이브러리를 앱에서 사용하기 위해서는 다운 받은 zip 속 <u>'naveridlogin_android_sdk'</u> 파일을 앱에 복사해주어야 한다.

  * 안드로이드 프로젝트의 **libs 폴더 밑**에 **'naveridlogin_android_sdk_4.2.6.arr' 파일**을 복사한다.
![복사](https://ifh.cc/g/A14HeQ.png)

<br>

👩 그렇다면 이제 build.gradle에 다음의 코드들을 추가한다.
```
// 네이버 아이디로 로그인
 implementation files('libs\\naveridlogin_android_sdk_4.2.6.aar')

 // 네아로 라이브러리
  implementation 'com.android.support:appcompat-v7:28.0.0'
  implementation 'com.android.support:support-core-utils:28.0.0'
  implementation 'com.android.support:customtabs:28.0.0'
  implementation 'com.android.support:support-v4:28.0.0'
```

<br>

👩여기까지 진행했다면 네이버 아이디로 로그인을 구현하기 위한 준비단계를 마친 셈이다!! 

로그인 구현부터는 다음 게시글에서 작성하도록 하겠다.

<br>

## 📃참고

👩 항상 다른 분들의 기술 블로그를 보며 학습을 진행했는데 이제는 공식 문서를 기준으로 나 스스로 이해하는 힘을 기르려고 한다!! 물론 다른 분들의 기술 블로그도 참고할 것이다...(세상은 넓고 나보다 똑똑한 똑똑이분들은 많다...) 느리더라도 조금씩 성장하자 아자아자!


* https://baessi.tistory.com/135
* https://developers.naver.com/docs/login/api/api.md
* https://lakue.tistory.com/49