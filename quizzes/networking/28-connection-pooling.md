# Quiz 28. Connection Pooling

Status: 답변 대기

## 핵심 확인

1. Connection Pooling이 필요한 이유를 설명하세요.
2. TCP/TLS Connection을 매 요청마다 새로 만들 때 어떤 비용이 발생하나요?
3. Pool에서 Connection을 빌리고 반환하는 기본 흐름을 설명하세요.
4. Pool Size가 너무 작으면 어떤 문제가 생기나요?
5. Pool Size가 너무 크면 왜 오히려 downstream을 망가뜨릴 수 있나요?
6. Connection Pool이 Backpressure 역할도 할 수 있는 이유는 무엇인가요?
7. Pool Acquisition Timeout은 왜 전체 Timeout Budget보다 작아야 하나요?
8. Idle Timeout과 Max Lifetime은 왜 필요한가요?
9. HTTP/1.1 Keep-Alive와 HTTP/2 Multiplexing이 Connection Pool 설계에 어떤 영향을 주나요?
10. App 20대, instance당 DB Pool 50개면 최대 몇 Connection이 가능한가요? 왜 이 계산이 중요한가요?
11. Connection Pool Exhaustion의 대표적인 원인은 무엇인가요?
12. Connection Leak을 C#에서 어떻게 줄일 수 있나요?

## 면접 꼬리 질문

13. DB가 갑자기 느려졌고 Pool active가 max, idle이 0입니다. 어떤 순서로 조사하겠습니까?
14. DB max_connections가 500인데 App가 20대라면 instance당 Pool을 무조건 25로 잡으면 충분한가요? 무엇을 더 봐야 하나요?
15. Pool을 두 배로 키웠더니 latency가 더 악화될 수 있는 이유는 무엇인가요?
16. Thread Pool 고갈과 Connection Pool 고갈이 서로 어떤 영향을 줄 수 있나요?
17. Connection Pool 관련 핵심 Metrics를 말해보세요.

## 60초 답변

18. "Connection Pooling이 무엇이고 Pool Size를 어떻게 생각해야 하나요?"에 60초 이내로 답하세요.
