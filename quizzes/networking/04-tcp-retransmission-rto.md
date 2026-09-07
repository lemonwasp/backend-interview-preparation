# 04. TCP Retransmission / RTO — 이해도 확인

상태: **답변 대기**

## A. 기본 개념

### Q1
TCP는 Packet Loss를 어떤 정보로 감지하나요?

### Q2
RTO란 무엇인가요?

### Q3
왜 RTO를 RTT와 같은 고정값으로 둘 수 없나요?

### Q4
Duplicate ACK는 어떤 상황에서 발생할 수 있나요?

### Q5
Fast Retransmit의 목적은 무엇인가요?

## B. 실무 연결

### Q6
Packet Loss 하나가 API Tail Latency를 크게 늘릴 수 있는 이유를 설명하세요.

### Q7
TCP Retransmission Timeout과 HTTP Client Timeout은 같은 개념인가요?

### Q8
TCP 재전송과 Application Retry가 동시에 존재할 수 있는 이유는 무엇인가요?

### Q9
여러 계층의 Retry가 중첩되면 어떤 문제가 생길 수 있나요?

## C. 기술면접

### Q10
TCP가 유실된 데이터를 복구하는 과정을 60초 이내로 설명하세요.

### Q11
"TCP를 쓰면 Packet Loss가 발생하지 않는다"는 말이 왜 틀렸나요?

### Q12
Packet Loss가 발생하면 재전송 외에 TCP의 어떤 제어 동작에 영향을 줄 수 있나요?

### Q13
평균 Latency뿐 아니라 p95/p99를 봐야 하는 이유를 Packet Loss와 연결해서 설명하세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 손실 복구 | ACK, RTO, 재전송 흐름을 설명하는가 |
| Fast Retransmit | Duplicate ACK의 의미를 이해하는가 |
| Timeout 구분 | TCP와 Application timeout을 구분하는가 |
| 성능 연결 | Loss와 Tail Latency를 연결하는가 |
