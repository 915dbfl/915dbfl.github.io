---
title: "[AlarmManager] 알림 구현하는 방식 고민(1)"
excerpt: "과연 어떤 방식으로 알림을 구현하는 것이 좋을까?"
categories:
  - Android
tag:
  - android
  - AlarmManager

last_modified_at: 2023-10-04
toc: true
toc_sticky: true
search: true
---

## 👩🏻‍💻 알림을 구현하고 싶은데?

사용자가 지정한 시간 10분 전에 이벤트를 받고 이를 처리하는 로직을 구현하고자 하는 상황이다.

이전에는 푸시 알림을 구현하고자 했을 때, <span style = "background-color:#fff5b1">FCM</span>을 가장 먼저 생각했다. 이번 프로젝트를 진행하면서 백엔드와 어떠한 방식으로 알림을 구현하는 것이 현재 우리의 요구사항에 가장 적합한지를 논의하였고, 그 결과 <sapn style = "background-color:#fff5b1">AlarmManager</span>을 사용하기로 결정하였다.

<br>

## 👩🏻‍💻 왜 FCM이 아니라 AlarmManager?

우선, 우리가 생각한 방식은 크게 두가지였다.

1. 서버 타이머 -> 사용자 설정 시간 10분 전 -> FCM으로 클라이언트에게 알림 전송

2. 클라이언트에서 AlarmManager를 사용

이때 FCM으로 처리할 경우, 발생하는 문제점이 크게 두 가지가 있다. 

1. 첫 번째로 각 사용자마다 존재하는 예약 주문에 대한 데이터를 서버에서 모두 확인하고 알림을 보내는 것은 부담이 있다.
2. fcm의 경우, <span style = "background-color:#fff5b1">네트워크 문제 등 다양한 요인으로 인해 알림이 정시에 보내진다는 보장이 없다.</span> 

예약 주문에 대한 알림은 정시에 보내지지 않는다면 의미가 없다. 따라서 FCM이 아닌 AlarmManager를 사용해 알림을 구현하기로 결정하였다. 다만, AlarmManager의 경우, 앱이 실행 중이지 않을 때도 알림을 보낼 수 있는 장점이 있다. 하지만 필자의 경우, 앱이 실행 중인 상황일 때만 알림을 보낼 것이기 때문에 앱 상태를 확인해 앱 실행 중이지 않을 경우에는 알림을 표시하지 않도록 구현할 예정이다.

다음 게시글에서는 Alarm 매니저를 사용하는 법에 대해 자세히 다루도록 하겠다.