# 06. UDP vs TCP

## 이번 학습 목표

- TCP와 UDP의 차이를 신뢰성·순서·연결 상태 관점에서 설명한다.
- UDP가 단순히 "빠른 TCP"가 아니라는 점을 이해한다.
- 어떤 애플리케이션이 TCP/UDP를 선택하는지 설명한다.

---

## 1. 가장 큰 차이

TCP는 **연결 지향적이고 신뢰성 있는 byte stream**을 제공합니다.
UDP는 **연결 설정 없이 datagram 단위로 보내는 단순한 전송 방식**입니다.

```text
TCP
- connection state
- ordered delivery
- retransmission
- flow control
- congestion control
- byte stream

UDP
- connectionless API model
- no built-in retransmission
- no built-in ordering
- message(datagram) boundary preserved
- small protocol overhead
```

UDP도 IP 위에서 동작하므로 packet loss, reorder, duplication이 발생할 수 있습니다.
필요한 신뢰성은 애플리케이션 계층이 직접 설계해야 합니다.

---

## 2. TCP는 무엇을 대신 해주는가?

애플리케이션이 TCP를 쓰면 Kernel TCP stack이 다음을 담당합니다.

- Sequence Number
- ACK
- Retransmission
- Ordering
- Flow Control
- Congestion Control

그래서 애플리케이션은 보통 "연결된 stream을 읽고 쓴다"는 추상화에 집중할 수 있습니다.

---

## 3. UDP는 왜 쓰는가?

UDP는 신뢰성을 포기하는 프로토콜이 아니라, **전송 정책을 애플리케이션이 더 많이 결정할 수 있게 하는 최소한의 전송 계층**에 가깝습니다.

예:

- 실시간 음성/영상
- 일부 게임 트래픽
- DNS 질의
- QUIC의 기반 전송

실시간 음성에서는 2초 늦게 도착한 옛 packet을 재전송받는 것보다 일부 손실을 버리고 최신 데이터를 받는 편이 나을 수 있습니다.

---

## 4. Message boundary

TCP는 byte stream입니다.

```text
send("ABC")
send("DEF")
```

수신 측이 반드시 `ABC`, `DEF` 두 번으로 읽는다는 보장은 없습니다.

```text
"ABCDEF"
```

으로 한 번에 읽거나 여러 조각으로 받을 수도 있습니다.

반면 UDP는 datagram 경계를 보존합니다. 애플리케이션이 보낸 한 datagram은 하나의 datagram 단위로 전달됩니다. 다만 datagram 자체가 유실될 수 있습니다.

---

## 5. UDP도 혼잡 제어를 해야 하지 않을까?

UDP 자체에는 TCP와 같은 혼잡 제어가 내장되어 있지 않습니다.
그렇다고 무제한으로 보내도 된다는 뜻은 아닙니다.

UDP 기반 프로토콜도 네트워크를 공정하고 안정적으로 사용하려면 애플리케이션 또는 상위 프로토콜 수준의 rate control / congestion control이 필요할 수 있습니다.

QUIC이 대표적인 예입니다. QUIC은 UDP 위에서 동작하지만 자체적으로 신뢰성, stream, congestion control을 구현합니다.

---

## 6. 백엔드 관점

일반적인 웹 API와 DB 연결은 TCP 기반이 많습니다. 이유는 요청·응답 데이터에서 순서와 신뢰성이 중요하기 때문입니다.

반면 latency가 중요하고 일부 손실을 허용하거나 프로토콜이 자체 신뢰성 메커니즘을 갖는다면 UDP가 적합할 수 있습니다.

프로토콜 선택은 단순히 "TCP는 느리고 UDP는 빠르다"가 아니라 다음 기준으로 봐야 합니다.

- 신뢰성이 필요한가?
- 순서가 필요한가?
- message boundary가 필요한가?
- connection state가 필요한가?
- 손실 시 재전송 정책을 누가 결정해야 하는가?

---

## 7. 60초 면접 답변

> TCP는 연결 지향적인 신뢰성 있는 byte stream을 제공하고, 순서 보장·재전송·흐름 제어·혼잡 제어를 프로토콜이 담당합니다. UDP는 연결 설정과 이런 신뢰성 기능을 기본 제공하지 않는 datagram 기반 전송입니다. 그래서 UDP가 단순히 더 빠른 TCP라기보다 애플리케이션이 재전송이나 지연 정책을 더 직접 결정할 수 있는 방식이라고 보는 편이 정확합니다. 웹 API처럼 데이터 정확성이 중요한 경우 TCP가 적합하고, 실시간 미디어처럼 오래된 데이터 재전송보다 최신성이 중요한 경우 UDP 계열 접근이 유리할 수 있습니다.

---

## 핵심 요약

```text
TCP = reliable ordered byte stream
UDP = lightweight datagram transport
UDP ≠ automatically faster in every workload
```

다음 주제: DNS
