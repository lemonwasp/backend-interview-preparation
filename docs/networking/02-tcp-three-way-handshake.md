# 02. TCP 3-way Handshake

## 이번 학습 목표

- TCP가 연결 설정 과정이 필요한 이유를 설명할 수 있다.
- SYN → SYN/ACK → ACK 흐름을 설명할 수 있다.
- Initial Sequence Number가 왜 필요한지 설명할 수 있다.
- Handshake 지연이 백엔드 Latency와 어떤 관계가 있는지 설명할 수 있다.

---

## 1. TCP 연결은 왜 바로 데이터를 보내지 않을까?

TCP는 신뢰성 있는 Byte Stream을 제공하기 위해 양쪽이 서로 통신 가능한지 확인하고,
각 방향의 Sequence Number 상태를 맞춰야 합니다.

따라서 데이터 전송 전에 연결 상태를 설정합니다.

```text
Client                      Server
  |                           |
  | -------- SYN -----------> |
  | <----- SYN + ACK -------- |
  | -------- ACK -----------> |
  |                           |
  |      ESTABLISHED          |
```

---

## 2. 첫 번째: SYN

Client가 연결을 시작합니다.

```text
SYN
Seq = x
```

여기서 `x`는 Client 측 Initial Sequence Number입니다.

의미는 대략:

> 나는 TCP 연결을 시작하고 싶고, 내 Sequence Number는 여기서 시작하겠습니다.

입니다.

---

## 3. 두 번째: SYN + ACK

Server는 Client의 SYN을 확인하고 자신의 연결 의사와 Sequence Number를 함께 보냅니다.

```text
SYN + ACK
Seq = y
Ack = x + 1
```

의미는:

> 네 SYN을 받았고, 나도 연결할 준비가 되었습니다. 내 Sequence Number는 y에서 시작합니다.

입니다.

---

## 4. 세 번째: ACK

Client는 Server의 SYN을 확인합니다.

```text
ACK
Ack = y + 1
```

이제 양쪽 모두 상대의 송수신 가능성과 초기 Sequence 상태를 확인했습니다.

TCP 연결은 ESTABLISHED 상태로 들어갑니다.

---

## 5. 왜 2-way가 아니라 3-way일까?

TCP는 양방향 통신입니다.

양쪽이 각각 다음을 확인해야 합니다.

- 상대가 내 메시지를 받을 수 있는가?
- 내가 상대 메시지를 받을 수 있는가?
- 양쪽 Initial Sequence Number를 서로 확인했는가?

3-way Handshake는 양방향 연결 상태와 Sequence Number 동기화를 확인합니다.

---

## 6. Sequence Number는 왜 필요한가?

TCP는 Packet이 아니라 **순서가 있는 Byte Stream**을 제공합니다.

네트워크에서는 Packet이:

- 유실될 수 있고
- 순서가 바뀔 수 있고
- 중복될 수 있습니다.

TCP는 Sequence Number를 사용해 데이터를 올바른 순서로 재조립하고 중복을 판단합니다.

---

## 7. 서버에서 무슨 일이 일어날까?

서버는 보통 Listening Socket을 만들어 대기합니다.

```text
socket
  ↓
bind
  ↓
listen
  ↓
Client SYN 도착
  ↓
TCP handshake
  ↓
연결 상태 생성
  ↓
accept
  ↓
Connected Socket
```

Handshake의 상당 부분은 Kernel TCP stack에서 처리됩니다.
애플리케이션이 SYN Packet 하나하나를 직접 처리하는 것은 아닙니다.

---

## 8. 연결 설정은 Latency다

새 TCP 연결에는 Network Round Trip이 필요합니다.

대략적으로 3-way Handshake는 Application Data 전송 전에 약 1 RTT의 연결 설정 비용을 추가합니다.

원격 서버와 RTT가 100ms라면 매 요청마다 새 TCP 연결을 만드는 것은 매우 비효율적일 수 있습니다.

그래서 다음 기술이 중요합니다.

- HTTP Keep-Alive
- Connection Pool
- Persistent Connection
- HTTP/2 Multiplexing

---

## 9. SYN Flood

공격자가 SYN만 대량으로 보내고 Handshake를 끝내지 않으면 Server의 연결 관련 자원을 소모시킬 수 있습니다.

이를 SYN Flood라고 합니다.

운영체제는 SYN cookies 같은 기법을 사용할 수 있습니다.

핵심은 TCP 연결 자체도 공짜가 아니라 Kernel state와 Queue 자원을 소비한다는 것입니다.

---

## 10. 자주 하는 오해

### “Handshake가 HTTP 연결이다”

아닙니다. TCP 연결 설정입니다. 그 위에서 HTTP가 동작합니다.

### “ACK는 Application이 데이터를 처리했다는 뜻이다”

TCP ACK는 해당 Byte가 TCP 계층에서 수신되었다는 의미이지, Application business logic 완료를 뜻하지 않습니다.

### “TCP 연결만 되면 TLS도 끝난다”

HTTPS에서는 TCP 연결 뒤 TLS handshake가 추가될 수 있습니다.

---

## 11. 60초 면접 답변

> TCP 3-way Handshake는 양쪽 Host가 서로 통신 가능한지 확인하고 Initial Sequence Number를 동기화해 신뢰성 있는 Byte Stream 상태를 만드는 과정입니다. Client가 SYN과 자신의 초기 Sequence Number를 보내고, Server는 SYN/ACK와 자신의 초기 Sequence Number를 보냅니다. 마지막으로 Client가 ACK를 보내면 연결이 ESTABLISHED됩니다. 이 과정은 새 연결마다 RTT 비용을 추가하기 때문에 백엔드에서는 Keep-Alive와 Connection Pool을 통해 연결 재사용이 중요합니다.

---

## 핵심 요약

```text
Client → SYN
Server → SYN + ACK
Client → ACK

목적 = 양방향 연결 확인 + Sequence Number 동기화
```

다음 주제: TCP 종료와 TIME_WAIT
