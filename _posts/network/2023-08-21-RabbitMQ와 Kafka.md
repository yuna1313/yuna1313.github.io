---
published: true
title: "RabbitMQ와 Kafka"
categories:
  - 네트워크
tags:
  - network
  - kafka
  - rabbitmq

toc: true
toc_sticky: true

date: 2023-08-21
last_modified_at: 2023-08-21
---

회사에서 프로젝트 설계 중 Message Queue에 대한 얘기가 나왔다. 
RabbitMQ와 Kafka 중 어떤 것을 사용하는 것이 더 효율적일지 정리 후 보고해달라 하셔서 알아보게 되었다.

일단, Kafka와 RabbitMQ의 정의를 알아보기 전 Message Queue(메시지 큐)에 대하여 알아야 한다.

---

## 1. Message Queue란?
- Message Queue는 시스템이 처리해야 할 일을 Producer가 차례대로 넣어 놓고 Consumer에 의해 하나씩 작업을 가져가 queue에서 삭제되는 통신 방법이다.
![message_queue](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/0c6666e4-f18e-4e1f-b428-fe9fc4495bdf)

## 2. RabbitMQ란?
RabbitMQ는 AMQP를 구현한 오픈소스 메시지 브로커이다. 여기서
### 메시지 브로커란?
![message_broker](https://github.com/yuna1313/yuna1313.github.io/assets/93983333/faa8a17c-2aa7-45bb-be91-c32b18e9a8b9)
- Publisher(Producer)로부터 전달받은 메시지를 Subscriber(Consumer)로 전달해주는 중간 역할이며 응용 소프트웨어 간에 메시지를 교환할 수 있게 한다. 이 때 메시지가 적재되는 공간을 Message Queue라고 하며 메시지의 그룹을 Topic이라고 한다.<br>
메시지 브로커는 Consumer가 데이터를 가져가면 queue에서 데이터가 삭제된다. 

### RabbitMQ의 차별화된 특징 
1. 다양한 클라이언트 라이브러리를 제공한다. (서로 다른 언어와 운영환경에서 데이터 공유)
2. 관리자가 UI Plugin과 Core Application을 구동하는데 40MB 미만의 메모리를 사용하여 가볍다.
3. 다양한 개발 툴을 지원한다.
4. 복잡한 라우팅을 지원한다.

## 3. Kafka란?
Kafka는 LinkedIn에서 개발한 고성능 분산 이벤트 스트리밍 플랫폼이다. Kafka는 이벤트 브로커에 속하는데 
### 이벤트 브로커란?
- 이벤트 브로커는 메시지 브로커의 기능 + 이벤트를 관리할 수 있다.<br>
메시지 브로커는 queue에서 데이터가 삭제되지만, 이벤트 브로커 같은 경우는 Publisher가 생산한 이벤트를 저장하여, Consumer가 특정 시점부터 이벤트를 다시 Consume할 수 있다. 

### Kafka의 차별화된 특징 
1. Producer와 Consumer가 분리되어있다. (분리되어있지 않은 경우, 어떤 서버의 응답이 늦어지면 해당 서버아 연결된 다른 서비스 서버들에게 이슈 발생)
2. Multi Producer Multi Consumer를 제공한다. (하나의 Topic에 여러 Producer와 Consumer가 접근 가능)
3. 디스크에 메시지를 저장한다. (이벤트 브로커 특성 상 이벤트를 바로 삭제하지 않고 보관 주기 동안 디스크에 저장)
4. 하나의 Kafka는 3개의 Broker로 시작하여 수십대 Broker까지 확장이 가능하다.

---

두 가지의 내용을 깊게 알아보기 보단, 특징을 위주로 살펴보았다.<br>
위와 같은 특징으로 보통 대기업에서 Kafka를 많이 사용하고 있다고 알려져 있는데, 찾아보니 꼭 그렇지만은 않은 것 같다.
그러니 프로젝트의 특징을 잘 고려하여 알맞는 것을 사용하길 바란다.

---

참조<br>
[https://heodolf.tistory.com/49](https://heodolf.tistory.com/49)<br>
[https://jaeyongchoi.tistory.com/11](https://jaeyongchoi.tistory.com/11)<br>
[https://wjjeong.tistory.com/57](https://wjjeong.tistory.com/57)<br>
[https://hongguri.tistory.com/41](https://hongguri.tistory.com/41)<br>