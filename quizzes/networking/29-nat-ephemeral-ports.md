# Quiz 29. NAT and Ephemeral Ports

Status: 답변 대기

## 핵심 확인

1. TCP Connection을 구분하는 4-tuple을 말하세요.
2. Ephemeral Port가 무엇인가요?
3. Client가 왜 outbound connection마다 Source Port를 필요로 하나요?
4. NAT의 역할을 설명하세요.
5. PAT/NAPT가 하나의 Public IP를 여러 내부 Connection에 공유하게 하는 방식을 설명하세요.
6. Ephemeral Port Exhaustion이 어떤 상황에서 발생할 수 있나요?
7. Connection Pooling이 Ephemeral Port pressure를 줄이는 이유는 무엇인가요?
8. TIME_WAIT과 높은 Connection Churn의 관계를 설명하세요.
9. NAT Gateway에서도 Connection 고갈이 발생할 수 있는 이유는 무엇인가요?
10. Server의 Listening Port와 Client의 Ephemeral Port를 구분해서 설명하세요.
11. NAT가 TCP reliability를 제공하지 않는 이유는 무엇인가요?
12. Outbound connection 장애 시 DNS 외에 어떤 계층을 점검해야 하나요?

## 면접 꼬리 질문

13. 외부 API 호출이 간헐적으로 connect 단계에서 실패합니다. 어떤 지표를 보겠습니까?
14. TIME_WAIT socket이 많습니다. 가장 먼저 OS tuning부터 하면 안 되는 이유는 무엇인가요?
15. App Instance 100대가 하나의 NAT Gateway를 공유할 때 Instance별 connection 수만 보면 안 되는 이유는 무엇인가요?
16. HTTP Client를 매 요청마다 새로 생성하는 패턴이 고트래픽에서 어떤 문제를 만들 수 있나요?
17. "Port는 65535개니까 Connection도 최대 65535개"라는 설명이 왜 부정확한가요?

## 60초 답변

18. "NAT와 Ephemeral Port 고갈이 Backend 장애에 어떻게 연결되나요?"에 60초 이내로 답하세요.
