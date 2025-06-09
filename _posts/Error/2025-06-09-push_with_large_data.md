---
title: "[Error] 대용량 변경사항 git push 오류"
categories:
  - Git
tag:
  - Git

last_modified_at: 2025-06-09
toc: true
toc_sticky: true
search: true
---

# 🫨 문제 상황

wequiz 프로젝트를 별도의 repository에서 refactoring을 하고자 기존 repository 코드를 푸시하려고 하는 상황에서 다음과 같은 오류가 발생하였다.

```shell
오브젝트 나열하는 중: 21649, 완료.
오브젝트 개수 세는 중: 100% (21649/21649), 완료.
Delta compression using up to 8 threads
오브젝트 압축하는 중: 100% (6326/6326), 완료.
error: RPC failed; HTTP 400 curl 22 The requested URL returned error: 400
send-pack: unexpected disconnect while reading sideband packet
오브젝트 쓰는 중: 100% (21649/21649), 5.01 MiB | 6.05 MiB/s, 완료.
Total 21649 (delta 8326), reused 21649 (delta 8326), pack-reused 0
fatal: the remote end hung up unexpectedly
Everything up-to-date
```

하지만 위와 같이 fatal에 적힌 오류 내용만을 가지고는 왜 remote 서버에서 예상치 못한 오류가 발생하는지 알 수 없었다.

<br>

# 👀 원인 파악

검색을 해본 결과 큰 데이터를 remote로 push할 때 git의 `http.postBuffer` 제한을 초과해 발생하는 문제였다.

## 1. git config 속 http.postBuffer에 대해 알아보자.

```text
http.postBuffer
Maximum size in bytes of the buffer used by smart HTTP transports when POSTing data to the remote system. For requests larger than this buffer size, HTTP/1.1 and Transfer-Encoding: chunked is used to avoid creating a massive pack file locally. Default is 1 MiB, which is sufficient for most requests.

Note that raising this limit is only effective for disabling chunked transfer encoding and therefore should be used only where the remote server or a proxy only supports HTTP/1.0 or is noncompliant with the HTTP standard. Raising this is not, in general, an effective solution for most push problems, but can increase memory consumption significantly since the entire buffer is allocated even for small pushes.
```

우선 위 내용을 정리하기 전에 우선 `http.postBuffer`가 어디에 활용이 되는지 짚고 넘어가자.

- http/1.1에서는 `connection: keep-alive` 헤더를 통해 여러 요청/응답을 하나의 TCP 연결을 통해 주고 받을 수 있다.
- 이때 대용량 데이터를 보내게 된다면 `Transfer-Encoding: chuncked`를 활용하여 하나의 데이터가 여러 개의 덩어리로 나뉘어 보내지게 된다.
- 이때 `http.postBuffer` 는 git에서 청크들이 전송되기 전에 메모리에 모아둘 수 있는 최대 데이터 크기를 정의하게 된다.

그렇다면 이 내용들을 기반으로 git config 속 내용을 정리해보자.

- 이러한 `http.postBuffer`는 기본적으로 1MiB로 이를 통해 대부분의 request를 처리할 수 있음
- 버퍼 사이즈를 늘릴 경우, 작은 push에 대해서도 늘린 크기 만큼의 buffer가 할당되기 때문에 메모리 소비가 증가할 수 있다.

## 2. 그렇다면 큰 데이터가 여러 개의 chunck로 분리돼 전송되고 있을까? 확인해보자.

```python
GIT_CURL_VERBOSE = 1
```

우선 [다음의 포스트](https://itchipmunk.tistory.com/636)를 통해 git이 http 통신을 위해 내부적으로 활용하는 curl 라이브러리 상세 출력을 활성화할 수 있다. 이를 활성화 하고 push를 진행한다면 push를 진행과 관련된 상세 출력을 확인할 수 있다.

```shell
18:30:34.672067 http.c:648              => Send header, 0000000205 bytes (0x000000cd)
18:30:34.672073 http.c:660              => Send header: GET .../info/refs?service=git-receive-pack HTTP/2
18:30:34.672074 http.c:660              => Send header: Host: github.com
18:30:34.672076 http.c:660              => Send header: User-Agent: git/2.39.1
18:30:34.672077 http.c:660              => Send header: Accept: */*
18:30:34.672078 http.c:660              => Send header: Accept-Encoding: deflate, gzip
18:30:34.672079 http.c:660              => Send header: Accept-Language: ko-KR, *;q=0.9
18:30:34.672081 http.c:660              => Send header: Pragma: no-cache
18:30:34.672082 http.c:660              => Send header:
```

위는 상세 출력 중 일부를 가져온 것이다. 공식 문서를 확인한다면 buffer 사이즈를 초과하는 데이터에 대해서는 http/1.1과 chunck가 활용된다고 나와 있지만 실제로는 http/2가 활용되고 있는 것을 확인할 수 있다.

이 부분에 대해 정확한 이유는 모르겠다. 하지만 예상되는 이유는 http/2에서는 http/1.1의 문제점을 해결하고 있으며, 최신 git과 curl 모두 기본적으로 http/2를 지원한다. 따라서 가능한 http/2를 활용하고자 내부적으로 동작하기 때문에 http/1.1이 활용되어야 하는 시점에서도 http/2가 활용되는 것이지 않을까 생각해본다.

## 3. 결론

결론적으로 <span style = "background-color:#fff5b1">큰 데이터를 remote로 전송할 때 http/2가 활용되고 chunck 형태로 데이터가 쪼개어 전송되지 못해</span> 다음과 같이 기본 버퍼 사이즈인 1MiB를 초과하는 데이터를 push하고자 할 때 오류가 발생하는 것이다.

<br>

# 🤔 해결

## 1. http/1.1 강제 해보기

그렇다면 git config를 통해 통신에 활용되는 http 버전을 1.1로 강제할 수는 없을까? 한번 시도해보자.

```shell
$ git config http.version HTTP/1.1 // http 버전 설정
$ git config --get http.version // 설정된 http 버전 확인
HTTP/1.1
```

그렇다면 이제 다시 push를 진행해보자!

```python
18:51:30.021649 http.c:660              => Send header: POST /.../git-receive-pack HTTP/1.1
18:51:30.021652 http.c:660              => Send header: Host: github.com
18:51:30.021655 http.c:660              => Send header: Authorization: Basic <redacted>
18:51:30.021658 http.c:660              => Send header: User-Agent: git/2.39.1
18:51:30.021661 http.c:660              => Send header: Accept-Encoding: deflate, gzip
18:51:30.021664 http.c:660              => Send header: Content-Type: application/x-git-receive-pack-request
18:51:30.021667 http.c:660              => Send header: Accept: application/x-git-receive-pack-result
18:51:30.021670 http.c:660              => Send header: Accept-Language: ko-KR, *;q=0.9
18:51:30.021673 http.c:660              => Send header: Transfer-Encoding: chunked

send-pack: unexpected disconnect while reading sideband packet
오브젝트 쓰는 중: 100% (21649/21649), 5.01 MiB | 5.43 MiB/s, 완료.
Total 21649 (delta 8326), reused 21649 (delta 8326), pack-reused 0
fatal: the remote end hung up unexpectedly
Everything up-to-date
```

다음과 같이 http/1.1과 chuncked가 활용되고 있음에도 불구하고, 여전히 전송이 되지 않는 것을 확인할 수 있다. 

관련해서 이유를 찾아본 결과, 위와 같은 방식이 오류를 발생시키는 원인은 크게 다음과 같았다.

1. 네트워크 안정성 문제
2. git 내부에 숨겨진 추가적인 제한
3. 프록시 서버에서 http1.1 지원 안함

정확한 원인은 무엇인지 파악하지는 못했지만, 네트워크는 안정적이었기에 아마 추가적인 제한이나 프록시 서버에서 해당 방식을 지원하지 않는 것이 문제이지 않을까 예상해본다.

## 2. `http.postBuffer 사이즈 늘리기`

http.postBuffer 사이즈를 필요한 만큼 늘려 전송하는 방식이 있다. push를 할 때 출력되는 값을 통해 전달하고자 하는 데이터가 어느 정도의 MiB인지를 파악할 수 있다. 

나의 경우, 5.01MiB 사이즈의 데이터를 전송해야 했기에 6MiB 만큼 버퍼 사이즈를 늘렸고, push 이후에는 다시 1MiB로 버퍼 사이즈를 재설정해주었다.

```shell
$ git config http.postBuffer 6291456 // 6MiB 
```

> 🚨 꼭 push를 완료한 후에는 1MiB로 버퍼 사이즈를 원상복귀시켜주자. 위에서 언급했듯이 버퍼 사이즈를 늘리게 된다면 불필요하게 큰 버퍼 사이즈를 매번 할당해야 하기 때문에 메모리 사용량이 증가할 수 있다.

## 3. `git ssh 사용하기`

SSH이 뭐길래 다른 설정 없이도 대용량 데이터를 안전하게 전송할 수 있는 걸까? SSH은 정처기를 공부할 때 원격 제어를 위해 사용되는 네트워크 프로토콜이라고만 학습했던 것이 기억에 남는다. 정확한 정의가 무엇인지를 확인하고 이를 활용해 어떻게 지금의 문제를 해결할 수 있는지 이해하고 넘어가자.

### 1. <span style = "background-color:#bcbcbc">SSH란?</span>

우선 SSH이란 Secure Shell Protocol의 약자로 네트워크 상의 `다른 컴퓨터에 로그인`하거나 `원격 시스템에서 명령을 실행`하고 `다른 시스템으로 파일을 복사`할 수 있도록 해주는 네트워크 프로토콜이다.

이를 위한 <span style = "background-color:#fff5b1">공개키 암호화 방식</span>을 활용해 강력한 인증 방법 및 보안을 제공한다. 개인키는 전달하지 않기 때문에 키가 유출될 위험이 적으며 이를 통해 불확실한 네트워크에서 안전하게 통신이 가능하도록 한다.

### 2. <span style = "background-color:#bcbcbc">git에서는 어떨 때 SSH가 필요할까?</span>

<img width="480" alt="Image" src="/assets/images/ssh.png" />

repository를 클론할 때 우리는 주로 Https에 존재하는 repository의 webURL을 활용하게 된다. 하지만 그 옆에 SSH 주소 역시 존재한다.

이 SSH를 활용하려면 우선 로컬에서 SSH 공개 / 개인 키를 생성하고 공개키를 repo 설정에 등록해줘야 한다. 이렇게 된다면 Git 클라이언트가 SSH 주소로 원격 서버에 접근을 시도하면, 로컬에 존재하는 개인키와 서버에 등록된 공개키를 통해 인증이 진행되게 된다. 이를 통해서 사용자가 인증을 위한 비밀번호나 토큰을 전송하는 과정이 필요 없어져 더욱 안전하게 통신이 가능해지게 된다. 

### 3. <span style = "background-color:#bcbcbc">git SSH와 대용량 데이터 전송</span>

위에서 확인했듯이 HTTPS를 활용한다면 대용량 데이터를 전송하기 위해 내부적으로 chunck, 버퍼 사이즈 등 제한이 존재한다. 하지만 SSH의 경우, 데이터를 연속적인 스트림 형태로 전달하기 때문에 대용량 데이터를 더욱 효율적으로 전송할 수 있게 된다.

> SSH를 설정해 활용하는 방법은 다양한 블로그에서 소개되고 있으니 이번 게시글에서는 다루지 않겠다.

<br>

# 👩🏻‍🔧 결론

그렇다면 이번 게시글의 핵심적인 내용을 정리하고 마무리해보고자 한다.

1. 대용량 변경사항을 push할 때는 postBuffer 사이즈를 늘려야 한다.
2. postBuffer 사이즈를 늘릴 경우, 작은 변경사항에 대한 push에 대해서도 늘린 buffer 사이즈 만큼 메모리를 사용하기 때문에 주의해야 한다.
3. ssh의 경우, 공개키 암호화 방식을 활용해 안전하게 다른 컴퓨터에 로그인하거나 파일 전송을 진행할 수 있다. 따라서 ssh를 활용해 push를 한다면 별다른 설정없이 대용량 데이터를 효율적으로 전송 가능하다.