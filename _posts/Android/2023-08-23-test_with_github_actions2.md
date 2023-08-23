---
title: "[CI] Github Action local property 처리"
excerpt: "Secrets 사용하기"
categories:
  - Android
tag:
  - android
  - github secrets

last_modified_at: 2023-08-23
toc: true
toc_sticky: true
search: true
---

[앞 게시글](https://915dbfl.github.io/android/test_with_github_actions/)에서 깃허브 액션을 달아보았다. 이번 게시글에서는 <b>local property</b>로 인해 발생하는 오류를 어떻게 해결하는지를 다룰 것이다.

# 👩🏻‍💻 Secrets

깃허브 액션을 통해 빌드를 하거나 테스트를 할 때, <b>프로젝트 안에서 local property</b>를 사용하면 해당 값을 찾을 수 없다는 오류가 발생한다.

필자는 서버 url과 같은 값을 안드로이드 프로젝트의 `local.properties` 파일에 저장해두었고, <span style = "background-color:#fff5b1">해당 파일은 버전 간리에서 제외가 되어 깃허브에 올라가지 않는다.</span> 따라서 해당 값을 직접 설정해주어야 빌드나 테스트 시 오류가 발생하는 것을 막을 수 있다.

## 🤔 설정 방법

> 깃허브 프로젝트 -> settings -> secrets and variables(actions) -> new repository secret

다음 과정을 통해 <span style = "background-color:#fff5b1">key, value</span>를 설정할 수 있다.

이를 설정하는 방법 [타블로그](https://velog.io/@2ast/Github-Github-actions%EC%97%90%EC%84%9C-Secrets%EB%A1%9C-%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0)에서 자세히 다루고 있으니 참고하기를 바란다!

## 🤔 활용 방법

- 접근
  ```kotlin
    $\{\{ secrets.ATT_BASE_URL }} # \제외하기
  ```

  이렇게 가져온 secret 값을 활용해 `local.properties` 파일을 생성해주어야 한다. <span style = "background-color:#fff5b1">파일의 이름은 local property가 저장된 파일 이름과 같게 설정해야 한다.</span>

- 파일 생성
  ```kotlin
      - name: generate local property
      run: |
        echo ATT_BASE_URL=\"$ATT_BASE_URL\" >> local.properties
        echo TMP_ACCESS_TOKEN=\"$TMP_ACCESS_TOKEN\" >> local.properties
        
      env:
        ATT_BASE_URL: $\{\{ secrets.ATT_BASE_URL }} # \제외하기
        TMP_ACCESS_TOKEN: $\{\{ secrets.TMP_ACCESS_TOKEN }} # \제외하기
  ```
  여기서 `>>`와 `>`는 다른 역할을 한다.
  * 같은 이름의 파일이 존재하는지 확인한다. 없다면 새로 생성!
  * 파일이 있다면 <span style = "background-color:#fff5b1">덮어쓰기? 추가하기?</span>
    * `>>`는 기존 파일에 새로운 내용 추가
    * `>`는 새로운 파일로 덮어쓰기

- 빌드 및 테스트 진행
  파일에 secret 값을 쓰는 것은 한 번만 진행하면 된다. 위와 같이 진행했다면 이제 빌드, 테스트 등은 별도의 secret을 정의해줄 필요 없이 그냥 진행하면 된다.

<br>

## 📃참고
* <https://docs.github.com/ko/actions/security-guides/encrypted-secrets>