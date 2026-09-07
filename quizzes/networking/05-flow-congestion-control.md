# 05. TCP Flow Control / Congestion Control — 이해도 확인

상태: **답변 대기**

## A. 기본 개념

### Q1
Flow Control과 Congestion Control의 목적 차이를 설명하세요.

### Q2
Receive Window(rwnd)는 무엇을 나타내나요?

### Q3
Congestion Window(cwnd)는 무엇을 제한하나요?

### Q4
왜 실제 전송 가능량을 `min(rwnd, cwnd)` 관점으로 생각할 수 있나요?

### Q5
Zero Window는 어떤 상황에서 발생할 수 있나요?

## B. 네트워크 동작

### Q6
Slow Start의 목적과 기본 동작을 설명하세요.

### Q7
Packet Loss가 cwnd에 영향을 줄 수 있는 이유는 무엇인가요?

### Q8
Bandwidth-Delay Product가 무엇이며 왜 중요한가요?

### Q9
Receiver가 매우 빨라도 Throughput이 낮을 수 있는 이유를 두 가지 말하세요.

## C. 백엔드 연결

### Q10
Application이 Socket에서 데이터를 너무 느리게 읽으면 TCP에 어떤 변화가 생길 수 있나요?

### Q11
Network congestion과 Application 처리 지연이 모두 "전송이 느리다"로 보일 수 있는 이유를 설명하세요.

### Q12
NIC가 1 Gbps라고 해서 항상 1 Gbps 가까운 Application Throughput이 나오지 않는 이유는 무엇인가요?

## D. 기술면접

### Q13
Flow Control과 Congestion Control을 60초 안에 비교해 설명하세요.

### Q14
"Flow Control이 있으니 Congestion Control은 필요 없지 않나요?"에 답하세요.

### Q15
"Slow Start는 이름 그대로 전송량을 천천히 선형 증가시키나요?"에 답하세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 목적 구분 | Receiver와 Network 보호를 분리하는가 |
| Window | rwnd와 cwnd 역할을 정확히 설명하는가 |
| 성능 | RTT/Loss/Window와 Throughput을 연결하는가 |
| 실무 연결 | Application read 속도와 Flow Control을 연결하는가 |
