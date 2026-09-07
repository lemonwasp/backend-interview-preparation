# 04. TCP Retransmission과 RTO

## 이번 학습 목표

- TCP가 Packet Loss를 어떻게 감지하고 복구하는지 설명할 수 있다.
- RTO와 RTT의 관계를 설명할 수 있다.
- Duplicate ACK와 Fast Retransmit을 설명할 수 있다.
- Packet Loss가 Latency에 왜 큰 영향을 주는지 설명할 수 있다.

---

## 1. TCP는 손실을 어떻게 복구할까?

IP는 Packet 전달을 보장하지 않습니다.

따라서 TCP는 Sequence Number와 ACK를 이용해 어떤 Byte가 정상 도착했는지 추적합니다.

```text
Sender                     Receiver
  | ---- Seq 1000 ------->   |
  | <---- ACK 1500 -------   |
```

ACK가 기대대로 오지 않으면 TCP는 손실 가능성을 판단하고 재전송합니다.

---

## 2. Timeout 기반 재전송

가장 기본적인 방법은 일정 시간 ACK가 오지 않으면 재전송하는 것입니다.

이 대기 시간이 RTO, Retransmission Timeout입니다.

```text
send segment
   ↓
wait for ACK
   ↓
RTO expires
   ↓
retransmit
```

RTO가 너무 짧으면 아직 정상적으로 오는 Packet을 중복 재전송할 수 있고,
너무 길면 실제 손실 발생 시 복구가 늦어집니다.

---

## 3. RTT와 RTO

RTT는 Round Trip Time, 즉 송신 후 응답이 돌아오기까지의 왕복 시간을 의미합니다.

TCP는 관측된 RTT를 바탕으로 네트워크 지연 변동까지 고려해 RTO를 추정합니다.

단순히:

```text
RTO = RTT
```

라고 고정하는 것이 아닙니다.

네트워크 지연은 계속 변하기 때문에 여유값이 필요합니다.

---

## 4. Duplicate ACK

중간 Segment가 유실되었다고 생각해봅시다.

```text
Seq 1      도착
Seq 2      유실
Seq 3      도착
Seq 4      도착
```

Receiver는 순서상 필요한 Seq 2를 기다리므로 같은 ACK를 반복해서 보낼 수 있습니다.

이 Duplicate ACK는 Sender에게:

> 뒤 데이터는 도착했지만 중간 데이터가 빠진 것 같다

는 힌트가 됩니다.

---

## 5. Fast Retransmit

TCP는 여러 Duplicate ACK가 관찰되면 RTO 만료를 기다리지 않고 유실 Segment를 빠르게 재전송할 수 있습니다.

이것이 Fast Retransmit의 핵심입니다.

```text
Duplicate ACKs
      ↓
loss inferred
      ↓
retransmit before timeout
```

정확한 세부 동작은 TCP 구현과 알고리즘에 따라 달라질 수 있습니다.

---

## 6. 재전송과 Congestion Control

Packet Loss는 단순한 데이터 유실 신호가 아니라 네트워크 혼잡의 신호일 수도 있습니다.

따라서 TCP는 손실이 발생하면 재전송만 하는 것이 아니라 전송 속도를 줄이는 Congestion Control 동작도 수행할 수 있습니다.

즉 Loss 하나가:

```text
재전송 비용
+
대기 시간
+
전송률 감소
```

를 함께 만들 수 있습니다.

---

## 7. Packet Loss가 Latency에 미치는 영향

API 요청이 작은 경우라도 핵심 TCP Segment 하나가 유실되면 응답 전체가 늦어질 수 있습니다.

특히 Tail Latency에서 손실의 영향이 큽니다.

예:

```text
평균 요청: 50ms
일부 요청: Packet loss → retransmission → 300ms+
```

그래서 평균 Latency만 보면 문제를 놓칠 수 있고 p95/p99 같은 Tail Latency도 중요합니다.

---

## 8. Application Timeout과 TCP RTO는 다르다

백엔드 Application에도 Timeout이 있습니다.

예:

- HTTP client timeout
- DB command timeout
- RPC deadline

이것은 TCP 내부 RTO와 별개입니다.

```text
Application timeout
      ≠
TCP retransmission timeout
```

Application timeout이 더 먼저 만료될 수도 있고, TCP는 내부적으로 계속 재전송을 시도할 수도 있습니다.

---

## 9. 재시도는 계층별로 중첩될 수 있다

문제가 발생했을 때 다음이 동시에 재시도할 수 있습니다.

```text
TCP retransmission
HTTP retry
Service retry
Queue retry
```

이런 중첩은 Retry Storm을 만들 수 있습니다.

따라서 Application Retry를 설계할 때 TCP가 이미 신뢰성 복구를 수행한다는 점과 별개로,
Application-level 실패 의미를 고려해야 합니다.

---

## 10. 자주 하는 오해

### “TCP는 Packet Loss가 없게 만든다”
아닙니다. Loss는 발생하며 TCP가 이를 감지하고 복구합니다.

### “ACK가 늦으면 무조건 Packet Loss다”
아닙니다. 지연, Queueing, Receiver 처리 상황 등 다양한 이유가 있습니다.

### “RTO는 항상 일정하다”
아닙니다. RTT 관측과 변동을 반영합니다.

### “TCP 재전송이 있으니 Application Retry는 필요 없다”
서로 다른 계층의 문제입니다. HTTP 503이나 Transaction 실패는 TCP 재전송으로 해결되지 않습니다.

---

## 11. 60초 면접 답변

> TCP는 Sequence Number와 ACK를 이용해 데이터 전달 상태를 추적하고, ACK가 일정 시간 오지 않으면 RTO가 만료된 것으로 보고 재전송합니다. RTO는 고정값이 아니라 관측된 RTT와 지연 변동을 기반으로 추정됩니다. 또한 중간 Segment가 유실되고 뒤 Segment들이 도착하면 Duplicate ACK가 반복될 수 있는데, TCP는 이를 통해 Timeout 전에 Fast Retransmit을 수행할 수 있습니다. Packet Loss는 재전송 대기뿐 아니라 Congestion Control로 전송률까지 낮출 수 있어서 Tail Latency에 큰 영향을 줄 수 있습니다.

---

## 핵심 요약

```text
Loss detection = Timeout + ACK pattern
RTO            = RTT 기반 동적 추정
Fast Retransmit= Duplicate ACK를 이용한 빠른 복구
```

다음 주제: Flow Control과 Congestion Control
