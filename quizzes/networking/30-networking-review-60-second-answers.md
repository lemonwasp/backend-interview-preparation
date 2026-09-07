# Quiz 30. Networking Review — Mock Interview

Status: 답변 대기

이 Quiz는 Networking 01~29 전체 복습용이다. 각 답변은 가능하면 60초 안에 말한다.

## A. Core

1. TCP와 UDP의 차이를 설명하세요.
2. TCP 3-way handshake가 왜 필요한가요?
3. TIME_WAIT은 왜 존재하나요?
4. TCP Retransmission과 Application Retry의 차이는 무엇인가요?
5. Flow Control과 Congestion Control의 차이는 무엇인가요?
6. DNS Resolver, Root, TLD, Authoritative Server의 역할을 설명하세요.
7. HTTP/1.1, HTTP/2, HTTP/3의 핵심 차이를 설명하세요.
8. QUIC이 UDP 위에서 동작하면서도 reliable할 수 있는 이유는 무엇인가요?
9. TLS가 제공하는 세 가지 핵심 보안 속성을 말하세요.
10. Certificate Chain은 어떻게 검증되나요?

## B. Infrastructure

11. Forward Proxy와 Reverse Proxy의 차이는 무엇인가요?
12. L4와 L7 Load Balancer를 비교하세요.
13. CDN과 HTTP Cache가 Origin 부하를 줄이는 방식을 설명하세요.
14. `no-cache`와 `no-store`를 비교하세요.
15. WebSocket과 SSE를 어떤 기준으로 선택하겠습니까?
16. REST와 gRPC를 어떤 기준으로 선택하겠습니까?
17. API Gateway와 Reverse Proxy의 차이를 설명하세요.

## C. Resilience

18. Rate Limiting과 Backpressure의 차이를 설명하세요.
19. Timeout Budget과 Deadline Propagation을 설명하세요.
20. Retry에 Exponential Backoff와 Jitter가 필요한 이유는 무엇인가요?
21. Retry Amplification은 어떻게 발생하나요?
22. Circuit Breaker와 Timeout의 차이는 무엇인가요?
23. Bulkhead가 연쇄 장애를 막는 방식을 설명하세요.
24. Idempotency Key가 필요한 대표적인 API를 하나 들고 처리 흐름을 설명하세요.

## D. Resource

25. Connection Pooling이 성능과 안정성에 어떤 영향을 주나요?
26. Pool Size가 너무 큰 경우 왜 문제가 되나요?
27. NAT와 Ephemeral Port를 설명하세요.
28. Connection Churn, TIME_WAIT, Ephemeral Port Exhaustion의 관계를 설명하세요.
29. Thread Pool과 Connection Pool의 차이를 설명하세요.
30. 무한 Queue가 왜 위험한가요?

## E. 장애 시나리오

31. 외부 API p99 latency가 갑자기 증가했습니다. 어떤 순서로 조사하겠습니까?
32. DB Connection Pool active가 max이고 idle이 0입니다. 가능한 원인과 대응을 설명하세요.
33. HTTP 503이 급증했습니다. Load Balancer부터 Application까지 어떤 지표를 보겠습니까?
34. outbound `connect()` 실패가 증가했습니다. DNS 외에 무엇을 확인하겠습니까?
35. 사용자가 결제 요청을 보낸 뒤 Timeout을 받고 다시 Retry했습니다. 중복 결제를 어떻게 막겠습니까?
36. downstream이 장애인데 Client, Gateway, Service 세 계층이 모두 3회 Retry합니다. 무슨 문제가 생기나요?
37. Consumer가 Producer보다 5배 느립니다. Queue를 크게 만드는 것만으로 해결할 수 없는 이유와 대안을 설명하세요.
38. WebSocket 서버를 1대에서 10대로 scale-out했습니다. 추가로 필요한 설계를 설명하세요.
39. HTTP/2 서비스에서 packet loss가 발생하면 다른 stream에도 영향이 갈 수 있는 이유는 무엇인가요?
40. NAT Gateway 뒤의 App 100대가 외부 API를 호출합니다. 각 App 지표는 정상인데 connect failure가 발생합니다. 무엇을 의심하겠습니까?

## 통과 기준

- 40개 중 32개 이상: 기본 통과 후보
- 비교 질문에서 핵심 축을 혼동하지 않음
- 장애 질문에서 관찰 지표 → 원인 가설 → 보호 조치 순서로 설명
- 틀린 항목은 해당 원문 문서로 돌아가 수정

최종 완료 여부는 이 점수만이 아니라 실제 답변 품질과 D+1 재시험을 포함해 판단한다.
