---
title: "[Android] Android keystore SHA-1"
excerpt: "debug/release"
categories:
  - Android
tag:
  - android keystore
  - SHA-1

last_modified_at: 2022-09-08
toc: true
toc_sticky: true
search: true
---

<br>

🙋‍♀️ 구글 로그인 기능을 사용할 때는 API 콘솔에 OAuth 클라이언트 ID, 즉 **인증정보**를 등록해주어야 한다.

여기서 잠깐!!

인증정보를 위한 SHA-1 인증서는 **Debug용, release용**으로 두 개 존재하며 두 인증서 모두 API 콘솔에 등록해주어야 한다.

<U>테스트 기기</u>에서 빌드할 때는 **Debug용**으로, <u>APK 파일</u>로 빌드할 때는 **Release용**으로 서명하게 된다.

따라서 ⭐스토어 출시하기 위해서는 Release용 SHA-1 인증서를 등록하지 않은 경우, 구글 로그인이 진행되지 않고 오류(**code 10**)가 발생하게 된다.⭐

지금부터 debug, release용 SHA-1를 추출하는 방법을 각각 정리해보고자 한다.

<BR>

# 👩 Android debug용 SHA-1 추출

* **Android Studio Gradle 뷰 -> 프로젝트 -> Tasks -> android -> signingReport**
<img src = "https://drive.google.com/uc?id=163qMTBVB3kOQWthF9dNZ9aJq4TljDO0Z" width = 500 height = 500>


<br>

# 👩 Android release용 SHA-1 추출
1. cmd를 통해 자바가 설치된 경로로 이동한다.
2. 다음 명령어를 실행한다.
```
keytool -list -v -keystore <jks path>
```
3. keystore 비밀번호를 입력하면 인증서 지문을 추출할 수 있다.

<br>

# 👩 API 콘솔에 인증서 등록하기
1. Google API Console에 들어가 인증정보를 사용자 인증 정보를 만든다.
<img src = "https://drive.google.com/uc?id=19op2CyxlTkLTfuU6mnGuArF2m8VDXRHQ" width = 500 height = 500>

2. 필요한 정보를 입력하고 키 등록을 마친다.
<img src = "https://drive.google.com/uc?id=1wZNUeau0Jof74v3SyIGhzWmre1GE3ndb" width = 500 height = 500>
* 패키지 이름: 해당 프로젝트 manifest 파일에서 확인
* 인증서 지문: SAH-1 서명 인증서 지문 입력

<BR>


  🙇‍♀️ 부족한 부분이 있다면 말씀해주세요! 감사합니다!

<br>

## 📃참고
* <https://featherwing.tistory.com/45>
* <https://it-banlim.tistory.com/19>
* <https://featherwing.tistory.com/55>
* <https://sgpassion.tistory.com/564>