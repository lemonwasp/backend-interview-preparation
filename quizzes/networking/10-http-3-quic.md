# 10. HTTP/3 & QUIC — 이해도 확인

상태: **답변 대기**

## A. 기본 개념

### Q1
HTTP/3가 TCP 대신 QUIC을 사용하는 핵심 이유를 설명하세요.

### Q2
QUIC이 UDP 위에서 동작한다고 해서 UDP와 같은 수준의 신뢰성만 제공하는 것이 아닌 이유는 무엇인가요?

### Q3
QUIC이 제공하는 기능을 네 가지 이상 말해보세요.

### Q4
HTTP/2의 TCP-level HOL Blocking을 다시 설명하세요.

## B. QUIC 구조

### Q5
QUIC의 독립적인 Stream이 HOL Blocking 영향을 어떻게 줄이나요?

### Q6
QUIC의 TLS 1.3 통합이 connection establishment에 어떤 장점을 주나요?

### Q7
0-RTT가 항상 안전한 것은 아닌 이유는 무엇인가요?

### Q8
Connection ID와 Connection Migration이 모바일 환경에서 왜 유용한가요?

### Q9
QUIC도 Congestion Control이 필요한 이유는 무엇인가요?

## C. 기술면접

### Q10
“HTTP/3는 UDP라서 HTTP/2보다 무조건 빠르죠?”라는 질문에 답하세요.

### Q11
HTTP/1.1 → HTTP/2 → HTTP/3의 발전을 connection reuse, multiplexing, HOL Blocking 관점에서 90초 이내로 설명하세요.

### Q12
외부 클라이언트가 HTTP/3를 사용한다고 해서 Backend Application Server도 반드시 HTTP/3를 사용하는 것은 아닌 이유를 설명하세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| QUIC | UDP 위의 별도 reliable transport임을 이해하는가 |
| HOL | HTTP/2 TCP HOL과 QUIC stream 격리를 연결하는가 |
| Handshake | TLS 1.3 통합과 0-RTT trade-off를 이해하는가 |
| Migration | Connection ID의 목적을 설명하는가 |
| 실무 | CDN/Proxy termination과 upstream protocol을 구분하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
