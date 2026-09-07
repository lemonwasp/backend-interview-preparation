# 05. TCP Flow Control과 Congestion Control

## 이번 학습 목표

- Flow Control과 Congestion Control의 목적을 구분할 수 있다.
- Receive Window와 Congestion Window의 역할을 설명할 수 있다.
- Sender의 실제 전송량이 두 제한의 영향을 받는 이유를 설명할 수 있다.
- Slow Start와 혼잡 신호의 기본 아이디어를 설명할 수 있다.

---

## 1. TCP Sender는 무한히 보내면 안 된다

TCP Sender가 빠르다고 무한히 데이터를 밀어 넣을 수는 없습니다.

두 가지 다른 한계가 있습니다.

```text
1. Receiver가 감당할 수 있는가? → Flow Control
2. Network가 감당할 수 있는가?  → Congestion Control
```

이 둘을 섞으면 안 됩니다.

---

## 2. Flow Control

Receiver의 Socket Receive Buffer가 가득 차면 더 이상 데이터를 안전하게 받을 수 없습니다.

Receiver는 자신의 여유 공간을 Window 정보로 Sender에게 알립니다.

이를 Receive Window, 흔히 `rwnd`라고 표현합니다.

```text
Receiver buffer 여유 큼
→ rwnd 큼
→ Sender가 더 많이 보낼 수 있음

Receiver buffer 여유 작음
→ rwnd 작음
→ Sender 전송 제한
```

목적은 **빠른 Sender가 느린 Receiver를 압도하지 않도록 하는 것**입니다.

---

## 3. Zero Window

Receiver buffer가 가득 차면 Receive Window가 0이 될 수 있습니다.

```text
rwnd = 0
```

Sender는 일반 데이터 전송을 멈추고 Receiver가 다시 Window를 열 때까지 기다려야 합니다.

이것은 Network Congestion과는 다른 문제입니다.

Application이 Socket에서 데이터를 너무 느리게 읽어도 Zero Window가 발생할 수 있습니다.

---

## 4. Congestion Control

Receiver는 충분히 빠르더라도 중간 Network가 혼잡할 수 있습니다.

Router Queue가 넘치거나 Packet Loss가 발생할 수 있습니다.

TCP는 네트워크 상황을 추정해 자신이 전송할 데이터 양을 제한합니다.

이를 위한 대표적인 상태가 Congestion Window, `cwnd`입니다.

목적은:

> 한 Connection이 네트워크 용량을 과도하게 사용해 전체 Network를 붕괴시키지 않도록 전송률을 조절하는 것

입니다.

---

## 5. 실제 전송 가능량

개념적으로 Sender가 ACK 없이 네트워크에 내보낼 수 있는 데이터 양은 두 Window의 영향을 함께 받습니다.

```text
usable window ≈ min(rwnd, cwnd)
```

Receiver가 병목이면 `rwnd`가 제한하고,
Network가 병목이면 `cwnd`가 제한합니다.

---

## 6. Slow Start

새 TCP 연결은 네트워크 용량을 정확히 모릅니다.

처음부터 엄청난 양을 보내면 혼잡을 일으킬 수 있습니다.

따라서 처음에는 비교적 작은 Congestion Window에서 시작해 ACK가 정상적으로 돌아오면 빠르게 Window를 늘려 네트워크 용량을 탐색합니다.

```text
small cwnd
   ↓ ACK success
larger cwnd
   ↓
larger cwnd
```

이 기본 아이디어가 Slow Start입니다.

이름은 Slow지만 초기 증가 자체는 빠르게 이루어질 수 있습니다.

---

## 7. 혼잡 신호가 오면

Packet Loss, ECN 등은 혼잡 신호로 활용될 수 있습니다.

TCP Congestion Control 알고리즘은 이런 신호가 나타나면 전송률을 줄입니다.

구체적인 증가/감소 방식은 Reno, CUBIC, BBR 등 알고리즘마다 다릅니다.

면접 기초 단계에서는 다음을 이해하는 것이 우선입니다.

```text
성공적인 전달 → 더 보낼 여지 탐색
혼잡 신호     → 전송량 감소
```

---

## 8. Bandwidth-Delay Product

대역폭이 높아도 RTT가 길면 충분한 데이터를 동시에 전송 중 상태로 유지해야 Link를 가득 사용할 수 있습니다.

이를 이해하는 데 Bandwidth-Delay Product(BDP)가 중요합니다.

개념적으로:

```text
BDP = bandwidth × RTT
```

네트워크 파이프 안에 동시에 존재할 수 있는 데이터 규모를 생각하는 데 도움이 됩니다.

Window가 지나치게 작으면 고속 장거리 Network를 충분히 활용하지 못할 수 있습니다.

---

## 9. Backend 관점

### 문제 1: Application이 Socket을 느리게 읽는다

```text
Application slow
   ↓
Receive Buffer fills
   ↓
rwnd shrinks
   ↓
Sender slows
```

이것은 Flow Control 쪽 현상입니다.

### 문제 2: Network Path가 혼잡하다

```text
Queue / loss / congestion signal
   ↓
cwnd reduced
   ↓
Throughput falls
```

이것은 Congestion Control 쪽입니다.

두 현상 모두 "전송이 느리다"로 보일 수 있지만 원인은 다릅니다.

---

## 10. HTTP 성능과 연결

큰 파일 다운로드나 Service-to-Service 통신에서는 다음이 Throughput에 영향을 줍니다.

- RTT
- Packet Loss
- Receive Window
- Congestion Window
- TCP Congestion Control algorithm
- Application read/write 속도

따라서 네트워크 속도는 단순히 NIC의 `1 Gbps` 숫자만으로 결정되지 않습니다.

---

## 11. 자주 하는 오해

### “Flow Control과 Congestion Control은 같은 것”
아닙니다. Receiver 보호와 Network 보호라는 서로 다른 문제입니다.

### “Receiver가 빠르면 항상 최대 속도로 전송된다”
아닙니다. Network congestion이 `cwnd`를 제한할 수 있습니다.

### “Slow Start는 항상 느리게 선형 증가한다”
아닙니다. 초기에는 Window를 빠르게 확장하며 용량을 탐색합니다.

### “대역폭만 높으면 전송 속도도 무조건 높다”
RTT, Loss, Window 크기 등도 중요합니다.

---

## 12. 60초 면접 답변

> TCP Flow Control은 빠른 Sender가 느린 Receiver의 Buffer를 넘치게 하지 않도록 Receive Window, rwnd를 이용해 전송량을 제한하는 기능입니다. 반면 Congestion Control은 중간 Network가 감당할 수 있는 수준으로 전송률을 조절하며 Congestion Window, cwnd를 사용합니다. 실제 전송 가능량은 개념적으로 두 Window 중 더 작은 값의 영향을 받습니다. 새 연결은 네트워크 용량을 모르기 때문에 Slow Start로 작은 cwnd에서 시작해 ACK를 보며 증가시키고, Loss 같은 혼잡 신호가 나타나면 전송량을 줄입니다. 따라서 Flow Control은 Receiver 보호, Congestion Control은 Network 보호라는 점이 핵심 차이입니다.

---

## 핵심 요약

```text
Flow Control      = Receiver 보호 → rwnd
Congestion Control= Network 보호  → cwnd
Sender limit      ≈ min(rwnd, cwnd)
```

다음 주제: UDP와 TCP 비교
