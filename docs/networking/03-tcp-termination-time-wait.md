# 03. TCP 종료와 TIME_WAIT

## 이번 학습 목표

- TCP 종료가 왜 보통 4-way 형태인지 설명할 수 있다.
- FIN과 ACK의 의미를 설명할 수 있다.
- TIME_WAIT가 왜 필요한지 설명할 수 있다.
- 대량 단기 연결에서 TIME_WAIT가 어떤 문제를 만들 수 있는지 설명할 수 있다.

---

## 1. TCP 연결은 양방향이다

TCP는 Full Duplex 연결입니다. 즉 양쪽 방향의 데이터 흐름을 각각 닫을 수 있습니다.

그래서 종료는 연결 시작보다 단계가 더 많아질 수 있습니다.

```text
Client                      Server
  |                           |
  | -------- FIN -----------> |
  | <------- ACK ------------ |
  | <------- FIN ------------ |
  | -------- ACK -----------> |
  |                           |
```

---

## 2. FIN은 무슨 뜻인가?

FIN은 대략 다음 의미입니다.

> 나는 이제 이 방향으로 더 보낼 데이터가 없습니다.

중요한 점은 FIN이 곧바로 양방향 연결 전체 종료를 의미하지 않는다는 것입니다.

한쪽이 FIN을 보내도 반대쪽은 아직 데이터를 보낼 수 있습니다.

이 상태를 Half-close 관점으로 이해할 수 있습니다.

---

## 3. 왜 4-way일까?

연결 시작에서는 Server가 SYN과 ACK를 한 Segment에 함께 보낼 수 있습니다.

종료에서는 상대방이 FIN을 받았다고 해서 즉시 자신도 보낼 데이터가 끝난 것은 아닐 수 있습니다.

따라서:

```text
FIN 수신
  ↓
ACK 먼저 전송
  ↓
남은 데이터 처리 가능
  ↓
자신도 종료 준비 완료
  ↓
FIN 전송
```

처럼 ACK와 FIN이 분리될 수 있습니다.

---

## 4. TIME_WAIT란?

일반적으로 Active Close를 수행한 쪽은 마지막 ACK를 보낸 뒤 일정 시간 TIME_WAIT 상태에 머무를 수 있습니다.

왜 바로 연결 정보를 버리지 않을까요?

### 이유 1: 마지막 ACK 유실 대비

마지막 ACK가 유실되면 상대는 FIN을 재전송할 수 있습니다.

TIME_WAIT 상태가 남아 있으면 다시 ACK를 보낼 수 있습니다.

### 이유 2: 오래된 중복 Segment가 새 연결에 섞이는 것 방지

같은 주소/Port 조합이 너무 빨리 재사용되면 이전 연결의 지연된 Segment가 새 연결 데이터로 오인될 위험이 있습니다.

TIME_WAIT는 이런 오래된 Segment가 네트워크에서 사라질 시간을 확보합니다.

---

## 5. 2MSL

TIME_WAIT 설명에서 흔히 2MSL이 등장합니다.

MSL은 Maximum Segment Lifetime의 개념적 값입니다.

대략:

```text
old segment가 갈 수 있는 시간
+
재전송 응답이 돌아올 수 있는 시간
```

을 고려해 일정 기간 기다립니다.

실제 OS의 TIME_WAIT 시간은 구현과 설정에 따라 다를 수 있습니다.

---

## 6. 누가 TIME_WAIT에 들어가나?

보통 **Active Close를 한 쪽**, 즉 먼저 FIN을 보내 종료를 주도한 쪽이 TIME_WAIT에 들어갑니다.

그래서 클라이언트가 매번 연결을 만들고 먼저 끊는 구조에서는 Client에 TIME_WAIT가 많이 쌓일 수 있습니다.

반대로 Server가 Active Close를 자주 수행하면 Server에 TIME_WAIT가 집중될 수 있습니다.

---

## 7. TIME_WAIT가 많으면 무조건 문제인가?

아닙니다. TIME_WAIT는 정상적인 TCP 안전장치입니다.

문제가 되는 경우는 예를 들어:

- 초당 매우 많은 짧은 연결 생성/종료
- Ephemeral Port 고갈
- Connection tracking 자원 압박
- FD나 다른 Kernel resource 문제와 함께 발생

등입니다.

따라서 TIME_WAIT 숫자만 보고 장애라고 단정하면 안 됩니다.

---

## 8. 백엔드에서는 어떻게 줄일까?

대표적인 방법은 연결을 재사용하는 것입니다.

- HTTP Keep-Alive
- Connection Pool
- Reverse Proxy upstream reuse
- HTTP/2 multiplexing

핵심은 TCP 안전장치를 억지로 없애는 것보다 **불필요한 새 연결 자체를 줄이는 것**입니다.

---

## 9. RST는 무엇인가?

정상적인 FIN 기반 종료와 달리 RST는 연결을 즉시 비정상 종료시키는 데 사용될 수 있습니다.

예를 들어 존재하지 않는 연결에 Segment가 오거나, Application이 연결을 강제로 중단하는 상황 등에서 나타날 수 있습니다.

`Connection reset by peer` 같은 오류는 RST와 관련될 수 있습니다.

---

## 10. 자주 하는 오해

### “FIN을 보내면 즉시 모든 통신이 끝난다”
아닙니다. 한 방향 송신 종료를 의미할 수 있습니다.

### “TIME_WAIT는 버그다”
아닙니다. TCP 안전성을 위한 정상 상태입니다.

### “Server만 TIME_WAIT가 생긴다”
아닙니다. 보통 Active Close를 수행한 쪽에 생깁니다.

### “TIME_WAIT를 0으로 만들면 성능이 좋아진다”
위험한 단순화입니다. 연결 재사용과 전체 연결 패턴을 먼저 봐야 합니다.

---

## 11. 60초 면접 답변

> TCP는 Full Duplex이므로 양쪽 송신 방향을 각각 종료할 수 있습니다. 일반적인 종료에서는 한쪽이 FIN을 보내고 상대가 ACK한 뒤, 상대도 데이터 전송이 끝나면 FIN을 보내고 마지막 ACK를 받는 4-way 형태가 됩니다. Active Close를 수행한 쪽은 마지막 ACK 유실 시 FIN 재전송에 대응하고, 이전 연결의 지연된 Segment가 새 연결에 섞이지 않도록 TIME_WAIT 상태에 일정 시간 머뭅니다. TIME_WAIT 자체는 정상이며, 문제가 된다면 대량의 짧은 연결 생성 같은 패턴을 확인하고 Keep-Alive나 Connection Pool로 연결 재사용을 고려합니다.

---

## 핵심 요약

```text
FIN = 이 방향으로 더 보낼 데이터 없음
TIME_WAIT = 마지막 ACK 재전송 대비 + 오래된 Segment 격리
```

다음 주제: Retransmission / RTO
