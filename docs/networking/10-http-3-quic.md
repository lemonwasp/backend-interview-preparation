# 10. HTTP/3 & QUIC

## 이번 학습 목표

- HTTP/3가 왜 TCP 대신 QUIC을 사용하는지 설명한다.
- QUIC의 stream multiplexing이 TCP-level HOL Blocking을 어떻게 줄이는지 설명한다.
- QUIC의 handshake, connection migration, congestion control 특성을 이해한다.

---

## 1. HTTP/3는 무엇이 달라졌나?

HTTP/2는 HTTP 계층에서 multiplexing을 도입했지만 여전히 하나의 TCP byte stream 위에서 동작했습니다.

HTTP/3는 **QUIC** 위에서 동작합니다.

```text
HTTP/1.1  → TCP
HTTP/2    → TCP
HTTP/3    → QUIC → UDP
```

QUIC은 UDP 위에 신뢰성, stream, congestion control, 보안 handshake 등을 구현하는 전송 프로토콜입니다.

---

## 2. 왜 UDP 위에 다시 많은 기능을 구현했을까?

TCP는 Kernel과 네트워크 장비에 깊게 자리 잡아 있어 프로토콜 변경과 배포가 느릴 수 있습니다.

QUIC은 UDP를 기반으로 많은 전송 로직을 user space에서 구현할 수 있어 비교적 빠르게 진화할 수 있습니다.

중요한 점은:

> QUIC = 단순 UDP

가 아니라는 것입니다.

QUIC은 자체적으로:

- reliable delivery
- stream multiplexing
- loss recovery
- congestion control
- connection management
- TLS 1.3 integration

을 제공합니다.

---

## 3. HTTP/2의 TCP-level HOL Blocking

HTTP/2에서 Stream A와 B는 논리적으로 독립적이어도 underlying TCP는 하나의 ordered byte stream입니다.

```text
TCP byte sequence
[A1][B1][A2][B2]
         X loss
```

중간 byte가 유실되면 뒤의 byte가 도착했어도 TCP는 순서를 맞추기 위해 재전송을 기다립니다.

그 결과 관련 없는 HTTP/2 stream까지 함께 지연될 수 있습니다.

---

## 4. QUIC Stream은 독립적이다

QUIC은 하나의 connection 안에 여러 독립적인 stream을 제공합니다.

```text
QUIC connection
├─ Stream A
├─ Stream B
└─ Stream C
```

Stream A의 데이터가 유실되어도 B와 C의 이미 도착한 데이터가 반드시 A 때문에 전달을 기다릴 필요는 없습니다.

따라서 TCP-level HOL Blocking의 영향 범위를 stream 수준으로 줄일 수 있습니다.

단, packet loss 자체가 사라지는 것은 아닙니다. 네트워크 혼잡과 congestion control 영향도 여전히 존재합니다.

---

## 5. TLS 1.3 통합

전통적인 HTTPS over TCP에서는 개념적으로:

```text
TCP handshake
   ↓
TLS handshake
   ↓
HTTP data
```

순서가 필요합니다.

QUIC은 TLS 1.3을 전송 handshake와 밀접하게 통합해 connection establishment latency를 줄입니다.

재연결에서는 조건이 맞으면 0-RTT data도 사용할 수 있습니다.

하지만 0-RTT는 replay risk 같은 보안 특성이 있으므로 모든 요청에 무조건 안전한 것은 아닙니다.

---

## 6. Connection ID와 Connection Migration

TCP connection은 일반적으로 source/destination IP와 port의 tuple에 강하게 연결됩니다.

모바일 환경에서:

```text
Wi-Fi → LTE/5G
```

처럼 IP가 바뀌면 기존 TCP connection이 끊길 수 있습니다.

QUIC은 Connection ID를 사용해 네트워크 경로가 바뀌어도 동일 connection을 이어갈 수 있는 기능을 제공합니다.

이것을 Connection Migration이라고 합니다.

---

## 7. QUIC도 Congestion Control을 한다

UDP 위에 동작한다고 해서 네트워크를 무제한 사용하지 않습니다.

QUIC은 자체 congestion control과 loss recovery를 구현합니다.

따라서:

```text
UDP = congestion control 없음
QUIC = UDP 기반이지만 congestion control 구현
```

으로 구분해야 합니다.

---

## 8. HTTP/3가 항상 더 빠른가?

아닙니다.

성능은 다음에 따라 달라집니다.

- RTT
- packet loss
- network path
- server/client implementation
- CPU overhead
- middlebox/firewall behavior
- connection reuse

특히 안정적인 low-latency network에서는 HTTP/2와 차이가 작을 수 있습니다.

HTTP/3의 장점은 단순 benchmark 숫자보다 **loss 환경에서 stream 격리, handshake 효율, connection migration** 같은 구조적 특성에 있습니다.

---

## 9. Backend 관점

HTTP/3를 직접 구현하지 않더라도 다음 구성 요소에서 접할 수 있습니다.

- CDN
- Reverse Proxy
- Load Balancer
- Browser-facing API
- Edge network

애플리케이션 서버 앞의 Proxy가 HTTP/3를 terminate하고 내부에서는 HTTP/2나 HTTP/1.1을 사용할 수도 있습니다.

따라서 외부 protocol과 내부 upstream protocol이 같다고 가정하면 안 됩니다.

---

## 10. 60초 면접 답변

> HTTP/3는 QUIC 위에서 동작하며 QUIC은 UDP를 기반으로 신뢰성, stream multiplexing, congestion control과 TLS 1.3 handshake를 구현합니다. HTTP/2는 여러 stream을 지원하지만 하나의 TCP ordered byte stream 위에 있기 때문에 packet loss가 발생하면 관련 없는 stream도 함께 막히는 TCP-level HOL blocking이 남습니다. QUIC은 stream별 전달 상태를 독립적으로 관리해서 한 stream의 loss가 다른 stream의 application delivery를 반드시 막지 않도록 합니다. 또한 handshake latency를 줄이고 Connection ID를 통해 IP가 바뀌는 모바일 환경에서 connection migration도 지원할 수 있습니다. 다만 HTTP/3가 모든 환경에서 무조건 더 빠른 것은 아닙니다.

---

## 핵심 요약

```text
HTTP/3 = HTTP semantics over QUIC
QUIC = reliable multiplexed transport over UDP
stream-level independence reduces TCP HOL impact
TLS 1.3 integrated
Connection ID enables migration
```

다음 주제:

- TLS / HTTPS
- Certificate
- Reverse Proxy / Load Balancer
