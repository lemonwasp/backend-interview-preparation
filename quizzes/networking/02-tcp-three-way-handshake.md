# 02. TCP 3-way Handshake — 이해도 확인

상태: **답변 대기**

## A. 기본 개념

### Q1
TCP 연결 전에 Handshake가 필요한 이유는 무엇인가요?

### Q2
SYN → SYN/ACK → ACK의 각 단계 의미를 설명하세요.

### Q3
Initial Sequence Number는 왜 필요한가요?

### Q4
TCP가 Byte Stream이라고 할 때 Sequence Number는 어떤 문제를 해결하나요?

## B. 실무 연결

### Q5
Listening Socket과 Connected Socket의 차이를 설명하세요.

### Q6
Handshake는 Application이 직접 처리하나요, Kernel이 처리하나요?

### Q7
새 TCP 연결을 매 요청마다 만들면 왜 느릴 수 있나요?

### Q8
Keep-Alive와 Connection Pool이 Handshake 비용을 어떻게 줄이나요?

### Q9
SYN Flood는 어떤 자원을 공격하나요?

## C. 기술면접

### Q10
TCP 3-way Handshake를 60초 안에 설명하세요.

### Q11
왜 2-way Handshake로는 부족한가요?

### Q12
TCP ACK를 받았다는 것이 Application 처리 완료를 뜻하지 않는 이유는 무엇인가요?

### Q13
HTTPS 연결에서 TCP Handshake 뒤에 추가로 어떤 과정이 필요할 수 있나요?

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 연결 목적 | 양방향 통신 확인과 Sequence 동기화를 말하는가 |
| 상태 흐름 | SYN/SYN-ACK/ACK를 정확히 설명하는가 |
| 계층 구분 | TCP ACK와 Application ACK를 구분하는가 |
| 실무 연결 | RTT와 연결 재사용을 연결하는가 |
