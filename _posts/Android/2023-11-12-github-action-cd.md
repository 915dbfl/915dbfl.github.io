---
title: "[github-action/CD] 안드로이드 CD 구축"
excerpt: "github-action + firebase를 활용한 안드로이드 CD 구축"
categories:
  - Android
tag:
  - Android
  - CI/CD
  - Firebase Appdistribution
last_modified_at: 2023-11-12
toc: true
toc_sticky: true
search: true
---

## 👩🏻‍💻 github-actions + Firebase AppDistribution
Firebase의 <span style = "background-color:#fff5b1">AppDistribution</span>는 
<span style = "background-color:#fff5b1">앱 출시 전 버전을 테스터에게 배포</span>하는 기능을 제공한다. 

다른 글을 보면 bitbucket, bitrise를 사용해 CI/CD pipeline을 구성하기도 한다. 하지만 필자는 이전 CI에서 사용했던 github-action을 이용해 CI/CD를 마저 구축해보고자 한다.

## 👩🏻‍💻 Firebase 설정
우선 필자는 해당 프로젝트에서 Firebase를 사용하지 않았었다. 그렇기 때문에 해당 앱과 firebase를 연동하는 작업부터 진행했다.

그 과정은 [Firebase 공식 문서](https://firebase.google.com/docs/android/setup?hl=ko)에 단계별로 자세히 나와 있으니 참고하길 바란다! 여기서 놓치지 말아야 할 점은 바로 <span style = "background-color:#fff5b1">`google-services.json`을 무조건 루트 디렉토리에 포함시켜야 한다는 점이다.</span>

그리고 추후, firebase 다른 제품들을 사용하기 위해서 아래 dependency를 미리 추가해 google-service.json 파일이 읽힐 수 있도록 하자!

```kotlin
// 루투 수준 gradle
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.android.library) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.navigation.safeargs) apply false
    alias(libs.plugins.dagger.hilt) apply false
    alias(libs.plugins.secret) apply false
    alias(libs.plugins.kotlin.serialization) apply false
    alias(libs.plugins.google.services) apply false //추가
}
```

```kotlin
//앱 수준 gradle
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("androidx.navigation.safeargs.kotlin")
    id("dagger.hilt.android.plugin")
    id("com.google.android.libraries.mapsplatform.secrets-gradle-plugin")
    id("com.google.gms.google-services") //추가
    kotlin("kapt")
}
```

이렇게 설정해두고 나중에 필요한 제품을 사용할 때마다 필요한 SDK를 추가해주면 된다.

## 👩🏻‍💻 Firebase CLI 설정하기
firebase appdistribution을 진행할 수 있는 방법은 무척 다양하다. 우리는 그 중에서 [`firebase CLI`](https://firebase.google.com/docs/app-distribution/android/distribute-cli?hl=ko&apptype=aab)를 사용할 것이다.

해당 앱이 google play store에 등록되어 있다는 가정하에 우리는 이전 단계에서 android app과 firebase 연동까지 진행했다. 이제 진행해야 할 부분은 <span style = "background-color:#fff5b1">Firebase Android 앱을 Google Play 개발자 계정에 연결</span>해야 한다.

### 1. Firebase Android 앱 - Google Play 개발자 계정 연결하기

![google play](/assets/images/google%20play.png)

프로젝트 설정 -> 통합에 들어가면 `Google Play 카드에서 연결`이 보일 것이다. 해당 카드를 선택해 연결을 진행하면 된다.

### 2. Firebase APP_ID 시크릿 설정

우선 github-action에서 firebase appdistribution을 진행하려면 <span style = "background-color:#fff5b1">firebase APP_ID, TOKEN</span>이 필요하다. 우선 APP_ID는 콘솔에서 쉽게 얻을 수 있다.

![app id](/assets/images/app_id.png)

프로젝트 설정에 들어가면 바로 해당 앱에 대한 앱 Id를 확인할 수 있다. 이를 github-action secret으로 등록해준다.

만약 github-action secret 등록하는 법이 궁금하다면 [다음 공식 홈페이지](https://docs.github.com/ko/actions/security-guides/using-secrets-in-github-actions)를 참고하길 바란다!

### 3. Firebase CLI 설정 및 Firebase Token 가져오기

[macOS에서 firebase CLI를 설치](https://firebase.google.com/docs/cli?hl=ko#install-cli-mac-linux)해보자!

```
curl -sL https://firebase.tools | bash
```

그렇다면 이제 firebase login을 진행하자.
```
firebase login
```

이렇게 로그인까지 진행했다면 project 목록을 확인해보자. 해당 목록에 내가 연결하고자 하는 앱이 잘 뜬다면 성공이다!
```
firebase projects:list
```

![firebase login](/assets/images/firebase_login.png)

그렇다면 이제 `firebase token`을 가져오자!

```
firebase login:ci
```

출력된 token을 app_id와 마찬가지로 github action secret으로 설정한다.

## 👩🏻‍💻 firebase의 appdistribution 사용 설정하기
이제 사용 준비를 모두 마쳤으니 firebase 콘솔 속 원하는 프로젝트에서 `firebase appdistribution` 사용을 시작해야 한다.

![app distribution](/assets/images/app_distribution.png)

다음과 같이 프로젝트에서 app distribution을 클릭하면 시작하기가 나온다. 시작하기를 클린한 후, 테스트 그룹을 설정해야 한다.

![test group](/assets/images/test_group.png)

그룹을 추가하면 그룹 옆에 <span style = "background-color:#fff5b1">작게 별칭이 나오고 해당 별칭을 명령줄 도구에서 그룹을 지정할 때 사용하라고 나와 있다.</span> 이를 사용하면 된다.

## 👩🏻‍💻 github action cd 구축하기

이제 모든 준비는 끝났다! yml파일에 cd 구축 코드를 추가해보자!

```
      - name: Build debug apk // 필자의 경우 debug apk 빌드
        run: ./gradlew assembleDebug --stacktrace

      - name: Upload apk to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }}
          token: ${{ secrets.FIREBASE_TOKEN }}
          groups: all-the-time
          file: app/build/outputs/apk/debug/app-debug.apk // debug apk 활용
          releaseNotes: |
            ${{ github.event.pull_request.title }}
            ${{ github.event.pull_request.html_url }}
            ${{ github.event.pull_request.body }}
```
- appId: 이전에 설정한 firebase app id 설정
- token: 이전에 설정한 firebase token 설정
- file: 필자의 경우, debug apk를 빌드해 활용하기 때문에 그에 맞는 파일 경로 설정
- releaseNotes: pull_request를 활용해 작성
  - 단 이를 활용하려면 workflow의 트리거가 pull_request를 할 때 진행해야 한다.
    ```
    on:
      pull_request:
    ```

## 👩🏻‍💻 마주한 오류들 정리
필자는 정말 많은 오류를 마주했었다.. 우선 파이어베이스 설정을 제대로 하지 않아 한참을 고생했고, 오타 때문에도 삽질을 꽤 많이 했다.. 혹시라도 같은 실수로 해당 오류를 마주하는 사람이 있다면 도움이 되고자 정리를 해본다!

### 1. firebase token issue
```
⚠  Authenticating with `FIREBASE_TOKEN` is deprecated and will be removed in a future major version of `firebase-tools`. Instead, use a service account key with `GOOGLE_APPLICATION_CREDENTIALS`: https://cloud.google.com/docs/authentication/getting-started
```
해석을 해보면 firebase_token이 deprecated 되었으니 google_application_credentials를 활용하라는 말이다. 

![token](/assets/images/token.png)

필자의 경우, google cloud에서 프로젝트에 대한 권한이 여러명 등록되어 있어 오류가 발생했었다. 소유자인 나를 제외한 나머지 구성원을 삭제하니 firebase token이 잘 인식되었다.

```
403에러
Error: failed to upload release. HTTP Error: 403, The caller does not have permission
https://stackoverflow.com/questions/62385911/firebase-app-distribution-failed-to-fetch-app-information-403-the-caller-do
```

해당 에러도 마찬가지이다. `IAM 및 관리자`를 확인해보자!

### 2. app_debug.apk 파일을 찾지 못한다?


1. 경로를 알맞게 작성했나?

    그렇다면 우선 file path가 잘 작성되었는지 확인하자. 만약 해당 path가 맞는지 잘 모르겠다면 android studio 툴을 활용해 직접 apk를 추출하고 경로를 확인하자!

    ![apk](/assets/images/apk.png)

    build variants 클릭 -> app variant를 debug로 설정

    ![app variant](/assets/images/app_variant.png)

    build -> build Bundles -> build APK(s)를 진행하면 빌등 완료 시 하단 팝업으로 locate을 확인할 수 있는 팝업이 뜨게된다. 

    ![locate](/assets/images/locate.png)

    폴더 속 app-debug.apk -> 우클릭 -> 정보 가져오기 -> 위치에서 우클릭 -> 경로 이름으로 복사를 진행해 경로를 확인한다.

2. yml 파일에서 upload를 진행하기 전에 해당 파일에 접근할 수 있는 환경을 셋팅해야 한다.

    필자의 경우, jdk 설정 / gradlew 퍼미션 변경 등의 작업을 진행하는 코드를 추가로 작성하였다. [깃 주소](https://github.com/AII-the-time/POS_Android/blob/develop/.github/workflows/android-cd.yml)를 참고하자!

    각각의 job은 별도로 실행되기 때문에 앞에서 아무리 셋팅 작업을 진행했어도 다시 셋팅을 진행해야 하는 것으로 이해했다.

## 👩🏻‍💻 회고
정말 간단한 작업인데 생각보다 많은 시간을 투자했다. 빨리 끝내고 싶은 마음에 급하게 진행하다 보니 잦은 실수들이 생겨 더 많은 시간을 잡아 먹은 것 같다. 그래도 이렇게 글을 작성해보니 내가 실수한 부분이 무엇이었는지를 확실히 알 수 있었고, 추후 계속해서 활용할 cd 과정을 잘 정리한 것 같아 뿌듯함이 든다.

cd 과정을 진행하면서 github에서 제공하는 draft pull_request에 대해서도 알 수 있었다. 처음에는 새로운 pr을 생성해 github action이 돌아가는 것을 확인했는데 draft pr을 통해서 업데이트된 commit을 포함해 같은 pr로 github action을 돌릴 수 있다는 점을 배울 수 있었다.

또한, workflow_run을 통해 workflow 관계를 정의하고 트리거하는 방법에 대해서도 이해하고 적용할 수 있었던 것 같다.