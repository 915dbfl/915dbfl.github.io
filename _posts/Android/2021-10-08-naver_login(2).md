---
title: "[Android/Login] 네이버 아이디로 로그인(2)"
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
toc_sticky: true
search: true
comment: true
---

## Intro
[[Android/Login] 네이버 아이디로 로그인(1)](https://915dbfl.github.io/android/naver_login(1)/)

👩 이전 과정에서 네아로를 사용하기 위한 준비는 마무리하였다.

그렇다면 지금부터는 네아로를 본격적으로 사용해보겠다.

저번과 마찬가지로 내가 참고한 문서는 [안드로이드 튜토리얼](https://developers.naver.com/docs/login/android/android.md)이다!!

<br>

🙋‍♀️ 구현하기에 앞서, 네아로를 구현하는 방법은 크게 두가지이다.
  * **<u>OAuthLoginButton 객체</u>** 사용 방법
  * **<u>OAuthLogin.startOAuthLoginActivity() 메서드</u>** 직접 실행 방법

나는 보다 간단한 첫번째 방식을 통해서 구현해보았다!!

<br>

그럼 한 번 달려보자~!!🏃‍♀️

<br>

## 👩 OAuthLoginButton 객체 추가: xml 수정

* 네아로를 사용하기 위해서 로그인을 진행할 버튼을 xml에 추가해야 한다.


```
<com.nhn.android.naverlogin.ui.view.OAuthLoginButton
      android:id="@+id/buttonOAuthLoginImg"
      android:layout_width="wrap_content"
      android:layout_height="50dp" />

```
* 해당 코드를 xml에 추가하게 되면 기본적인 네이버 아이디로 로그인 버튼이 생성된다.

<br>

xml 수정은 여기서 끝이다!!
## 👩 초기화 과정

네아로 라이브러리를 애플리케이션에 적용하기 위해서는 다음 과정을 통해서 **네아로 인스턴스를 받아와 초기화를 진행해줘야 한다.**


👩 우선 먼저 앱을 등록해 얻은 클라이언트 아이디 값과 클라이언트 시크릿 값과 클라이언트 네임을 변수에 저장한다.

```
  // 클라이언트 정보
  val OAUTH_CLIENT_ID = "클라이언트 아이디"
  val OAUTH_CLIENT_SECRET = "클라이언트 시크릿"
  val OAUTH_CLIENT_NAME = "애플리케이션 이름"
```

이렇게 작성한 **클라이언트 정보**를 네아로 인스턴스 **초기화** 과정에서 설정해준다.

```
mOAuthLoginModule = OAuthLogin.getInstance();
mOAuthLoginModule.init(
	OAuthSampleActivity.this
	,OAUTH_CLIENT_ID
	,OAUTH_CLIENT_SECRET
	,OAUTH_CLIENT_NAME
);
```

🙋‍♀️ 또한, 네아로 로그인을 통해 로그인이 진행된 후, 종료되었을 때 실행할 **OAuthLoginHandler 클래스를 핸들러로 등록**해주어야 한다.

```
binding.buttonOAuthLoginImg.setOAuthLoginHandler(mOAuthLoginHandler)
```

<br>

## 👩 OAuthLoginHandler 구현
벌써 마지막 단계이다!!

* 마지막으로 진행해야 할 점은 로그인 버튼에 적용한 핸들러를 구현하는 것이다.

```
  private val mOAuthLoginHandler: OAuthLoginHandler = object : OAuthLoginHandler() {
      override fun run(success: Boolean) {
          if (success) {
            //필자의 경우, 접근 토큰만을 받아왔다.
              ACtoken = mOAuthLoginInstance.getAccessToken(this@LoginActivity).toString()
            // 로그인 후, 다음 액티비티 진행 설정
              redirectSignupActivity()
          } else {
              val errorCode = mOAuthLoginInstance.getLastErrorCode(this@LoginActivity).code
              val errorDesc = mOAuthLoginInstance.getLastErrorDesc(this@LoginActivity)
              Toast.makeText(
                  this@LoginActivity,
                  "errorCode:$errorCode, errorDesc:$errorDesc",
                  Toast.LENGTH_SHORT
              ).show()
          }
      }
  }

```

다음과 같은 코드를 추가한다면 필요에 따라서 접근 토큰, 갱신 토큰 등을 받아올 수 있다.


## 👩‍💻 activity 전체 코드
```
class LoginActivity : AppCompatActivity() {
    private lateinit var binding: LoginBinding

    // 클라이언트 정보
    val OAUTH_CLIENT_ID = "..."
    val OAUTH_CLIENT_SECRET = "..."
    val OAUTH_CLIENT_NAME = "..."

    lateinit var mOAuthLoginInstance: OAuthLogin

    lateinit var ACtoken: String
    lateinit var nextIntent: Intent

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = LoginBinding.inflate(layoutInflater)
        val view = binding.root
        Log.i("view", view.toString())
        setContentView(view)

        initData()
    }

    //초기화 단계
    private fun initData() {
        mOAuthLoginInstance = OAuthLogin.getInstance()
        mOAuthLoginInstance.showDevelopersLog(true)
        mOAuthLoginInstance.init(this, OAUTH_CLIENT_ID, OAUTH_CLIENT_SECRET, OAUTH_CLIENT_NAME)
        binding.buttonOAuthLoginImg.setOAuthLoginHandler(mOAuthLoginHandler)
    }

    //핸들러 구현
    private val mOAuthLoginHandler: OAuthLoginHandler = object : OAuthLoginHandler() {
        override fun run(success: Boolean) {
            if (success) {
                ACtoken = mOAuthLoginInstance.getAccessToken(this@LoginActivity).toString()
                redirectSignupActivity()
            } else {
                val errorCode = mOAuthLoginInstance.getLastErrorCode(this@LoginActivity).code
                val errorDesc = mOAuthLoginInstance.getLastErrorDesc(this@LoginActivity)
                Toast.makeText(
                    this@LoginActivity,
                    "errorCode:$errorCode, errorDesc:$errorDesc",
                    Toast.LENGTH_SHORT
                ).show()
            }
        }
    }

    private fun redirectSignupActivity(){
        nextIntent = Intent(this, MainActivity::class.java)
        startActivity(nextIntent)
    }
}
```

---

이로써 로그인 구현은 끝이다! 생각보다 간단하지 않은가!🙃

조금 더 자세한 방식이 궁금하다면 **공식 홈페이지에서 제공하는 [네아로 예제 깃허브 코드](https://github.com/naver/naveridlogin-sdk-android/tree/master/naveridlogin_android_sdk_sample)**를 참고하길 바란다.
비록 로그아웃, 토큰 갱신과 같은 세부 기능에 대한 것은 작업하지 않았지만 우선은 로그인 구현 부분은 끝이다!
추후에 프로젝트를 진행하며 자세한 사항들을 추가 정리하도록 하겠다!!

이번 게시글은!

이만!!

끝! ~(˘▾˘~)

<br>

## 📃참고

* https://baessi.tistory.com/135
* https://developers.naver.com/docs/login/api/api.md
* https://lakue.tistory.com/49