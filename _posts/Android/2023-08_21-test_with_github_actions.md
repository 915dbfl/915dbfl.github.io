---
title: "[CI] Github Action 달기"
excerpt: "+ test 실행 및 리포트 얻기"
categories:
  - Android
tag:
  - android
  - github-action

last_modified_at: 2023-08-21
toc: true
toc_sticky: true
search: true
---

# 👩🏻‍💻 [github action](https://docs.github.com/ko/actions)이 뭘까?
| 리포지토리에서 바로 <b>소프트웨어 개발 워크플로를 자동화, 사용자 지정 및 실행한다.</b> CI/CD를 포함하여 원하는 작업을 수행하기 위한 작업을 검색, 생성 및 공유하고 완전히 사용자 정의된 워크플로에서 작업을 결합할 수 있다.

<br>

# 👩🏻‍💻 [yml 파일](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

프로젝트에서 github actions를 추가하게 되면 자동으로 `.github/workflows` 디렉토리에 `.yml` 파일이 생성되며, 다음의 코드가 작성되어 있는 것을 확인할 수 있다.

```kotlin
# workflow의 이름으로 원하는 형식으로 수정이 가능하다.
name: Android CI

# workflow가 실행될 때 컨트롤 하는 부분
# 해당 workflow는 main branch에 push나 pull_request 이벤트가 일어날 때 trigger된다.
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

# 각각의 job들은 독립적으로 실행이 된다.
# 여기서 해당 job은 main에 puah, pull_request가 될 때 빌드가 제대로 되는지를 확인해주는 역할이다.
jobs:
  build: # job의 이름을 원하는 형식으로 지정해준다.

    runs-on: ubuntu-latest # job이 실행되는 runner 타입 지정

    # 각 step은 순서대로 실행이 된다.
    steps: 
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    # 필요한 jdk 세팅 작업
    - uses: actions/checkout@v3
    - name: set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        cache: gradle

    # gradlew 파일의 실행 권한을 부여
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
    # 권한이 부여된 gradlew를 활용해 빌드 진행
    - name: Build with Gradle
      run: ./gradlew build
```
아마 이를 그대로 사용하면 build 실패가 발생할 수도 있을 것이다. <b>여기서 <span style = "background-color:#fff5b1">jdk 버전</span>을 프로젝트에 맞게 변환해주어야 한다.</b>

<br>

# 👩🏻‍💻 github actions를 활용한 test 

그렇다면 빌드에서 나아가 github actions를 활용해 test를 실행해보자! 기본적으로 가장 많이 작성되는 unitTest를 실행하는 코드를 작성할 것이다.

```kotlin
  test:
    needs: build # build에 종속성을 갖게 된다. 즉, build job이 끝난 후 실행된다.
    runs-on: macos-latest # 필자는 mac을 사용하고 있으므로 macos 환경에서 실행되도록 한다.
    steps:

    # 위에서 말했듯이 각각의 job은 독립적이다. 따라서 checkout과 jdk 설정을 별도로 다시 진행한다.
      - uses: actions/checkout@v3
      - name: set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: 'zulu'
    
    # Unit Test를 실행
      - name: Unit Test
        run: ./gradlew testDebugUnitTest

    # test에 대한 report upload
      - name: Upload Test Reports Folder
        uses: actions/upload-artifact@v2
        if: ${{ always() }}
        with:
          name: reports
          path: app/build/test-results 
```

여기서 업로드된 테스트 리포트를 눈으로 확인할 수 있도록 출력하는 코드도 추가한다. 여기서 나는 marketplace에서 제공하는 `Android Test Report Action`을 사용할 것이다.

```kotlin
  # report를 출력하는 job을 추가한다.
  report:
    runs-on: ubuntu-latest
    needs: test # test에 대한 report를 출력하는 것이므로 test job에 종속을 가진다.
    if: ${{ always() }}
    steps:
      - name: Download Test Reports Folder
        uses: actions/download-artifact@v2
        with:
          # with를 통해 액션의 입력 매개변수 이름 지정
          # reports라는 이름의 artifact를 다운로드 하게 된다.
          name: reports

      - name: Android Test Report
        uses: asadmansr/android-test-report-action@v1.2.0
```

그렇다면 이제 결과를 확인해보자!!

<img src = "https://drive.google.com/uc?id=1LSCRPnlZobsZaXMx1pYWa2XEeQrAxkeW">

우선, 종속성에 따라 job들이 이어진 형태를 확인할 수 있을 것이다! 그리고 만약 모든 job들이 성공적으로 build된다면 아래와 같이 test에 대한 report를 확인할 수 있을 것이다.

<img src = "https://drive.google.com/uc?id=15wgRkF8U3TFBQPcwmNxgT70QsJh41H_J">

# 🤔 추가정리

위 코드를 작성하게 되면 다음과 같은 warning이 발생하였다.

| test: The following actions uses node12 which is deprecated and will be forced to run on node16: actions/upload-artifact@v2. For more info: https://github.blog/changelog/2023-06-13-github-actions-all-actions-will-run-on-node16-instead-of-node12-by-default/


| report: The following actions uses node12 which is deprecated and will be forced to run on node16: actions/download-artifact@v2. For more info: https://github.blog/changelog/2023-06-13-github-actions-all-actions-will-run-on-node16-instead-of-node12-by-default/

즉, <b>node12가 deprecate되었으므로 node16을 사용하라</b>는 말이다! 따라서 `v2`를 `v3`로 변경함으로써 위의 warning을 제거할 수 있었다.

```kotlin
...
      - name: Unit Test
        run: ./gradlew test

      - name: Upload Test Reports Folder
        uses: actions/upload-artifact@v3 # ✓ 수정
        if: ${{ always() }}
        with:
          name: reports
          path: app/build/reports/tests

  report:
    runs-on: ubuntu-latest
    needs: test
    if: ${{ always() }}
    steps:
      - name: Download Test Reports Folder
        uses: actions/download-artifact@v3 # ✓ 수정
        with:
          name: reports

      - name: Android Test Report
        uses: asadmansr/android-test-report-action@v1.2.0
```

<br>

github action을 적용하던 중, `local `


# 😌 정리

github action을 미루고 미루다 드디어 적용을 하였다. 막상 적용해보니 정말 간단했고, 내가 정의한 대로 개발 워크플로가 자동화되는 것이 매우 흥미로웠던 것 같다. 처음에는 뭐가 잘못됐는지 몰라 계속해서 build 오류가 발생했지만 각각의 의미를 알고 원하는 코드를 추가하여 워크플로우가 성공적으로 빌드되는 것을 확인하니 무척 뿌듯했다. 앱 개발을 어느 정도 마무리하고 CD까지 github actions를 통해 진행하여 추후 블로그에 내용을 정리해야겠다. :0

<img src = "https://drive.google.com/uc?id=1S9GExz2qmykvE19qm47FtVAqmDbq4LqL">

<br>

## 📃참고
* <https://github.com/marketplace/actions/android-test-report-action>
* <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions>
