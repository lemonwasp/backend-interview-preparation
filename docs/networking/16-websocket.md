# 16. WebSocket

## 학습 목표

이 문서를 읽고 나면 다음을 설명할 수 있어야 합니다.

- WebSocket이 왜 필요한가
- HTTP와 어떤 관계인가
- Full-duplex가 무엇인가
- 연결을 오래 유지할 때 서버에 어떤 비용이 생기는가
- WebSocket과 SSE를 언제 구분해 쓰는가

---

## 1. 가장 쉬운 설명

HTTP 요청/응답은 기본적으로 Client가 요청을 보내면 Server가 응답하는 구조입니다.

하지만 채팅, 실시간 게임, 협업 편집처럼 서버와 클라이언트가 서로 자주 메시지를 보내야 한다면 매번 새 HTTP 요청을 만드는 방식은 불편합니다.

WebSocket은 한 번 연결을 만든 뒤 그 연결을 계속 유지하면서 양쪽이 자유롭게 메시지를 주고받게 해 줍니다.

즉,

> WebSocket = 하나의 오래 유지되는 양방향 통신 채널

입니다.

---

## 2. Full-duplex란?

Full-duplex는 양쪽이 서로 독립적으로 송신할 수 있다는 뜻입니다.

예를 들어 채팅에서는:

- Client가 메시지를 보낼 수 있고
- Server도 별도 요청을 기다리지 않고 새 메시지를 Client에게 밀어줄 수 있습니다.

이 점이 일반적인 HTTP request/response 모델과 가장 큰 차이입니다.

---

## 3. 연결 시작

WebSocket은 보통 HTTP/1.1 요청으로 시작해서 WebSocket 프로토콜로 전환합니다.

대표적인 흐름:

```text
Client
  │ HTTP Upgrade request
  ▼
Server
  │ 101 Switching Protocols
  ▼
WebSocket connection
```

대표 Header:

```text
Connection: Upgrade
Upgrade: websocket
```

이후에는 일반 HTTP 요청/응답 형식이 아니라 WebSocket frame 단위로 통신합니다.

---

## 4. 왜 Polling보다 나은가?

Polling은 Client가 주기적으로 묻습니다.

```text
새 메시지 있어?
없음
새 메시지 있어?
없음
새 메시지 있어?
있음
```

문제:

- 불필요한 요청이 많음
- Header 비용 반복
- 실시간성이 polling interval에 묶임

WebSocket은 연결을 유지하므로 서버가 데이터가 생겼을 때 즉시 보낼 수 있습니다.

---

## 5. 하지만 WebSocket도 공짜가 아니다

연결 하나를 오래 유지한다는 것은 서버가 연결 상태를 계속 관리한다는 뜻입니다.

연결 수가 커질수록 다음이 중요합니다.

- Socket/File Descriptor 수
- Kernel socket buffer
- Application connection state
- heartbeat/ping-pong
- timeout
- load balancer idle timeout
- connection draining

즉 10만 명이 접속하면 단순히 HTTP request 10만 개가 아니라 장기 연결 10만 개를 관리할 수 있습니다.

---

## 6. Ping / Pong과 연결 생존 확인

TCP 연결이 존재해 보이더라도 중간 NAT, Proxy, Load Balancer에서 연결이 끊겼을 수 있습니다.

그래서 WebSocket에서는 ping/pong 또는 application-level heartbeat를 사용해 연결이 살아 있는지 확인합니다.

하지만 heartbeat 주기를 너무 짧게 잡으면 트래픽과 CPU 비용이 커집니다.

---

## 7. Scale-out에서 중요한 문제

Server A와 WebSocket 연결이 맺어졌다고 합시다.

```text
Client ── WebSocket ── Server A
```

그런데 다른 요청으로 들어온 메시지를 Server B가 처리한다면?

```text
Publisher → Server B
Client    → Server A
```

Server B가 Server A에 연결된 Client에게 직접 보낼 수 없습니다.

그래서 대규모 시스템에서는 다음 구조를 자주 사용합니다.

```text
Server A ─┐
Server B ─┼─ Redis Pub/Sub / Kafka / Message Broker
Server C ─┘
```

즉 WebSocket scale-out은 단순 Load Balancing만으로 끝나지 않습니다.

---

## 8. Sticky Session은 해결책인가?

같은 Client를 항상 같은 Server로 보내면 연결 상태 관리가 쉬워질 수 있습니다.

하지만 단점:

- 특정 서버 쏠림
- 장애 시 재연결 필요
- Scale-out 유연성 감소

따라서 sticky session만 믿기보다 connection state와 message distribution을 별도로 설계하는 것이 중요합니다.

---

## 9. WebSocket vs HTTP

| 항목 | HTTP request/response | WebSocket |
|---|---|---|
| 통신 | 요청 중심 | 양방향 |
| 연결 | 재사용 가능하지만 요청 단위 | 장기 연결 |
| Server Push | 제한적 | 자연스러움 |
| 상태 관리 | 상대적으로 단순 | 연결 상태 관리 필요 |
| 적합 | 일반 API | 채팅, 실시간 게임, 협업 |

---

## 10. WebSocket vs SSE

핵심:

- WebSocket: 양방향
- SSE: Server → Client 단방향

Client가 Server에 자주 실시간 메시지를 보내야 하면 WebSocket이 적합합니다.

Server가 Client에게 이벤트만 계속 밀어주면 SSE가 더 단순할 수 있습니다.

---

## 11. Backend 사례

채팅 서비스:

```text
Client A ─┐
Client B ─┼─ WebSocket Gateway ─ Message Broker ─ Chat Service
Client C ─┘
```

여기서 WebSocket Gateway의 역할은 비즈니스 로직 전체를 처리하는 것이 아니라 장기 연결과 message routing에 집중시키는 편이 확장에 유리할 수 있습니다.

---

## 12. 자주 하는 오해

### 오해 1: WebSocket은 TCP를 대체한다

아닙니다. 일반적인 WebSocket은 TCP 위에서 동작합니다.

### 오해 2: WebSocket이면 무조건 빠르다

아닙니다. 실시간 양방향 통신에 적합한 모델일 뿐이며 연결 관리 비용이 있습니다.

### 오해 3: 서버가 여러 대여도 Load Balancer만 두면 끝난다

장기 연결과 cross-node message delivery 문제를 별도로 고려해야 합니다.

---

## 13. 60초 면접 답변

> WebSocket은 하나의 장기 연결 위에서 Client와 Server가 양방향으로 메시지를 주고받을 수 있게 하는 프로토콜입니다. 보통 HTTP Upgrade로 연결을 시작한 뒤 WebSocket frame으로 통신합니다. Polling보다 불필요한 요청과 지연을 줄일 수 있어 채팅이나 실시간 협업에 적합하지만, 연결 수가 증가하면 File Descriptor, socket buffer, heartbeat, idle timeout과 같은 자원 관리가 중요해집니다. 여러 서버로 확장할 때는 특정 Client가 어느 서버에 연결되어 있는지와 서버 간 메시지 전달을 고려해야 해서 Redis Pub/Sub이나 Kafka 같은 메시지 계층을 함께 사용할 수 있습니다.
