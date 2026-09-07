# 04. Message Queue — Interview Drills

상태: **답변 대기**

## Questions

1. Message Queue를 사용하는 핵심 이유를 설명하세요.
2. 긴 이미지 변환 작업을 synchronous HTTP 대신 queue 기반으로 설계하면 어떤 장점이 있나요?
3. At-most-once와 At-least-once의 차이는 무엇인가요?
4. 왜 실무에서 `at-least-once + idempotent consumer` 조합이 중요한가요?
5. Consumer acknowledgement는 언제 보내는 것이 일반적으로 안전한가요?
6. Retry를 무한히 하면 왜 위험한가요?
7. Dead Letter Queue의 목적은 무엇인가요?
8. Queue에서 ordering을 어떻게 다룰 수 있나요?
9. Queue backlog가 늘어난다고 consumer 수를 무작정 늘리면 왜 위험한가요?
10. Queue depth 외에 어떤 운영 지표를 함께 봐야 하나요?
11. Queue와 Pub/Sub의 기본적인 차이를 설명하세요.
12. DB write와 message publish 사이에는 어떤 dual-write 문제가 있나요?
13. Transactional Outbox가 이 문제를 어떻게 줄이나요?
14. Poison Message란 무엇이며 어떻게 다루나요?
15. 작은 CRUD 서비스에 Kafka를 무조건 넣으면 안 되는 이유는 무엇인가요?

## Evaluation Criteria

- Queue를 latency/decoupling/spike absorption 관점에서 설명하는가
- delivery semantics와 idempotency를 연결하는가
- retry/DLQ/order/backpressure를 설명하는가
- consumer scaling을 downstream capacity와 연결하는가
- Outbox/CDC와 dual-write 문제를 이해하는가
- queue 자체가 복잡도와 eventual consistency를 추가한다는 점을 말하는가
