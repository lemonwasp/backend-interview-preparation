# 11. TLS와 HTTPS — 이해도 확인

상태: **답변 대기**

## A. 핵심 개념

### Q1
HTTPS는 HTTP와 무엇이 다른가요?

### Q2
TLS가 제공하는 세 가지 핵심 보안 성질을 설명하세요.

### Q3
TLS에서 비대칭 암호와 대칭 암호를 함께 사용하는 이유는 무엇인가요?

### Q4
TLS 1.3 Handshake를 ClientHello부터 Application Data까지 순서대로 설명해보세요.

## B. 오해 확인

### Q5
“HTTPS를 쓰면 네트워크에서 아무 Metadata도 볼 수 없다”는 설명이 왜 틀렸나요?

### Q6
“TLS는 암호화만 제공한다”는 설명이 왜 부족한가요?

### Q7
왜 매 HTTP Request마다 새 TCP/TLS 연결을 만드는 것이 비효율적일 수 있나요?

## C. 실무 연결

### Q8
Connection Pool과 Keep-Alive가 TLS 관점에서도 중요한 이유를 설명하세요.

### Q9
TLS Termination이 무엇이며 어디에서 수행할 수 있나요?

### Q10
Load Balancer에서 TLS를 종료하고 Backend까지 평문 HTTP를 쓰는 설계의 장단점을 말해보세요.

## D. 기술면접

### Q11
“HTTPS가 어떻게 안전한가요?”에 60초 이내로 답해보세요.

### Q12
“대칭키가 빠르다면 처음부터 대칭키만 쓰면 되지 않나요?”에 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 보안 목표 | Confidentiality / Integrity / Authentication을 구분하는가 |
| 키 사용 | 인증·키 합의와 대칭 암호의 역할을 구분하는가 |
| 성능 | Handshake와 Connection Reuse 비용을 설명하는가 |
| 시스템 설계 | TLS Termination의 장단점을 말할 수 있는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
