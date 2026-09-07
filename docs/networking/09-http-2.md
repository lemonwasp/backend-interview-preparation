# 09. HTTP/2

## 이번 학습 목표

- HTTP/2가 HTTP/1.1의 어떤 병목을 개선했는지 설명한다.
- Stream multiplexing과 Binary Framing을 설명한다.
- HTTP/2에서도 TCP-level HOL Blocking이 남는 이유를 설명한다.

---

## 1. HTTP/2의 핵심 변화

HTTP/2는 HTTP의 의미론을 크게 바꾸지 않고 전송 방식을 개선했습니다.

```text
HTTP semantics
GET /users
Status 200
Headers
Body
```

이런 기본 의미는 유지하면서 한 TCP connection 안에서 여러 stream을 동시에 처리할 수 있게 했습니다.

---

## 2. Binary Framing

HTTP/1.1은 사람이 읽기 쉬운 text 기반 메시지 형식입니다.

HTTP/2는 메시지를 binary frame으로 나눕니다.

```text
Connection
├─ Stream 1
│  ├─ HEADERS frame
│  └─ DATA frame
├─ Stream 3
│  ├─ HEADERS frame
│  └─ DATA frame
└─ Stream 5
```

여러 stream의 frame이 같은 TCP connection 위에서 interleave될 수 있습니다.

---

## 3. Multiplexing

HTTP/1.1에서는 하나의 연결에서 요청/응답 순서 제약 때문에 병렬성이 제한되었습니다.

HTTP/2에서는 각각의 요청이 독립적인 stream을 가집니다.

```text
Stream A: Request A ───── Response A
Stream B: Request B ─ Response B
Stream C: Request C ───────── Response C
```

A가 느려도 B의 application-level response를 먼저 진행할 수 있습니다.

이것이 HTTP/1.1 pipelining의 application-level HOL Blocking을 크게 줄입니다.

---

## 4. 그런데 HOL Blocking이 완전히 없어졌나?

아닙니다.

HTTP/2는 여전히 TCP 위에서 동작합니다.
TCP는 하나의 ordered byte stream입니다.

예를 들어 TCP Segment 하나가 유실되면:

```text
TCP bytes
[1][2][3][4][5]
       X loss
```

뒤의 byte가 도착했더라도 TCP는 순서대로 애플리케이션에 전달해야 하므로 잃어버린 부분의 재전송을 기다릴 수 있습니다.

그 위의 HTTP/2 stream이 서로 독립적이어도 underlying TCP stream 하나가 막히면 여러 HTTP/2 stream이 함께 영향을 받을 수 있습니다.

이것이 **TCP-level HOL Blocking**입니다.

---

## 5. Header Compression

HTTP 요청은 반복되는 Header가 많습니다.

예:

```text
Host
User-Agent
Accept
Cookie
Authorization
```

HTTP/2는 HPACK을 사용해 Header 압축을 제공합니다.

목표는 반복되는 Header 전송 비용을 줄이는 것입니다.

---

## 6. 하나의 Connection을 오래 쓰는 장점

HTTP/2는 일반적으로 하나의 origin에 대해 적은 수의 TCP connection으로 여러 요청을 multiplexing할 수 있습니다.

장점:

- TCP/TLS handshake 감소
- connection 수 감소
- FD / kernel state 감소
- connection별 congestion control 분산 문제 완화

하지만 하나의 connection에 많은 stream이 몰리면 그 connection 장애나 packet loss의 영향 범위가 커질 수도 있습니다.

---

## 7. Stream과 TCP Connection의 차이

HTTP/2 Stream은 애플리케이션 계층의 논리적 통신 단위입니다.

TCP connection 자체를 여러 개 만든 것이 아닙니다.

```text
HTTP/2 Stream 1 ┐
HTTP/2 Stream 3 ├─ one TCP connection
HTTP/2 Stream 5 ┘
```

따라서 stream마다 별도 TCP handshake나 congestion control state가 있는 것이 아닙니다.

---

## 8. Server Push

HTTP/2에는 Server Push 기능도 정의되어 있습니다.

서버가 클라이언트 요청 전에 관련 resource를 미리 보낼 수 있는 개념입니다.

하지만 실제 웹 생태계에서는 기대만큼 널리 유용하지 않았고 브라우저 지원도 축소되었습니다.

따라서 HTTP/2의 핵심을 Server Push라고 설명하는 것보다 **multiplexing + binary framing + header compression**을 중심으로 설명하는 것이 안전합니다.

---

## 9. Backend 관점

HTTP/2는 특히 다음에서 중요합니다.

- gRPC
- API Gateway
- Reverse Proxy
- Service-to-service communication
- many concurrent requests

gRPC가 HTTP/2를 활용하는 이유 중 하나도 multiplexed stream과 binary framing을 활용하기 좋기 때문입니다.

---

## 10. 60초 면접 답변

> HTTP/2는 HTTP semantics는 유지하면서 binary framing과 stream multiplexing을 도입해 HTTP/1.1의 연결 사용 효율을 개선했습니다. 하나의 TCP connection에서 여러 독립적인 HTTP stream의 frame을 섞어 전송할 수 있어서 HTTP/1.1 pipelining의 application-level HOL blocking을 줄입니다. 또한 HPACK으로 header compression을 제공합니다. 하지만 HTTP/2는 TCP 위에서 동작하므로 TCP segment가 유실되면 ordered byte stream 특성 때문에 여러 HTTP/2 stream이 함께 영향을 받을 수 있는 TCP-level HOL blocking은 남습니다. HTTP/3는 이 문제를 줄이기 위해 QUIC을 사용합니다.

---

## 핵심 요약

```text
HTTP/2
= binary framing
+ multiplexed streams
+ HPACK

HTTP/1.1 HOL 개선
but TCP-level HOL remains
```

다음 주제: HTTP/3 & QUIC
