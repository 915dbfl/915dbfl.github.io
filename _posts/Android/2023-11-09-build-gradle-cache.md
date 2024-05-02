---
title: "[github-action/빌드 속도 최적화] Build gradle cache를 통산 빌드 시간 단축"
excerpt: "github-actions 빌드 시간 장난 아닌데...?"
categories:
  - Android
tag:
  - Android
  - CI
  - Build Gradle Cache
last_modified_at: 2023-11-09
toc: true
toc_sticky: true
search: true
---

## 👩🏻‍💻 늘어나는 빌드 시간...

<img src = "https://drive.google.com/uc?id=1zPWnHOom0IpSUdSEYb4PkrxXhIFt4wNZ">

github-action을 통해 CI를 구축한 후, 빌드를 기다리는 시간이 너무 길게 느껴졌다. 이를 조금이라도 단축할 순 없을까 고민하던 중, `build gradle cache`에 대해 알게 되어, 이를 적용하는 방법을 다뤄보고자 한다.

## 👩🏻‍💻 Build Gradle Cache가 필요한 이유

### Gradle은 캐싱을 사용한다! 그런데.. github-actions에서는 적용이 안된다고?
프로젝트의 규모가 커질수록 빌드시간이 늘어나는 이유는 당연하게도 <span style = "background-color:#fff5b1">빌드 때 의존성 패키지를 모두 다운받기 때문이다.</span>

<span style = "background-color:#fff5b1">`Gradle`은 의존성을 캐싱하는 방식을 통해 첫 빌드 이후에는 캐시에 저장된 파일을 재사용한다.</span>

하지만 <span style = "background-color:#fff5b1">github-actions의 workflow는 매번 새로운 환경에서 실행</span>되기 때문에 마치 첫 빌드와 같이 모든 의존성을 다운받게 된다.

### github-action에서 적용하는 캐싱을 사용하자!
github-action에서도 반복적으로 사용되는 의존성에 대해 `캐싱`하는 기능을 제공한다. 이를 위해 우리는 <span style = "background-color:#fff5b1">actions/cache</span>을 사용할 수 있다.

## 👩🏻‍💻 actions/cache 사용하기

```kotlin
    - name: Gradle cache
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*') }}
        restore-keys: |
          ${{ runner.os }}-gradle-
```
path: 캐시가 되는 파일 runner내 파일 경로
key: 캐시 저장, 복원에 사용되는 키
restore-key: 위에 설정한 key로 cache miss가 발생할 때 사용하는 후보키들

## 👩🏻‍💻 결과 확인하기

그렇다면 이제 결과를 비교해보자! 캐시가 적용되지 않은 상태에서 처음 캐시를 적용하는 경우 아래와 같이 

<img src = "https://drive.google.com/uc?id=15w_2SNrjuqAMmwkyJsDfAql0r9MCERA4">

restore할 캐시가 없다 뜨고, 마지막 캐시 작업을 완료하는 것을 확인할 수 있다. 이때 빌드가 진행되는 Unit test job을 본다면 약 <span style = "background-color:#fff5b1">6분</span>가량 소요됐다.

![결과](/assets/images/locate.png)

캐시를 적용한 후다. <span style = "background-color:#fff5b1">cache restored</span>가 성공적으로 이뤄지고, 마지막으로 cache hit이 일어난 것까지 확인된다. 이때 Unit test는 약 <span style = "background-color:#fff5b1">2분</span>이 소요되었다. 즉, 약 66% 속도 향상이 된 것이다!

간단한 코드지만 이를 통해 전반적인 개발 시간이 줄어든 것 같아 뿌듯하다!

## 👩🏻‍💻 참고
- <https://devjem.tistory.com/76>
- <https://studynote.oopy.io/e3c94599-a07d-4241-98a3-7cda0040e3a3>
- <https://sungbin.land/github-actions-%EC%9C%BC%EB%A1%9C-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-ci-cd-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-1aaaa6595c4a>