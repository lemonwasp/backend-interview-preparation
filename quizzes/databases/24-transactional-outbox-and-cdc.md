# Quiz — Transactional Outbox / CDC

Status: **답변 대기**

## Questions

1. DB 저장 + Kafka publish를 단순히 순서대로 실행하면 어떤 dual-write 문제가 생기나요?
2. Transactional Outbox의 핵심 아이디어는 무엇인가요?
3. Business Row와 Outbox Row를 같은 Transaction에 넣는 이유는 무엇인가요?
4. Outbox Relay가 Event를 중복 발행할 수 있는 시나리오를 설명해보세요.
5. 왜 Consumer Idempotency가 필요한가요?
6. At-least-once delivery란 무엇인가요?
7. Broker가 exactly-once 기능을 제공해도 전체 비즈니스 Side Effect가 exactly-once라고 단정할 수 없는 이유는 무엇인가요?
8. CDC란 무엇인가요?
9. Log-based CDC는 어떤 종류의 DB Log를 이용할 수 있나요?
10. Outbox + CDC를 함께 쓰는 구조를 설명해보세요.
11. Event Ordering이 중요한 예를 하나 들어보세요.
12. 같은 Aggregate의 Event 순서를 지키기 위해 어떤 정보를 활용할 수 있나요?
13. Outbox Table이 계속 커질 때 어떤 운영 문제가 생기나요?
14. Outbox가 해결하지 못하는 문제를 3개 말해보세요.
15. Search Index나 Cache 동기화에 CDC가 어떻게 활용될 수 있나요?

## Interview Drill

> "주문 DB 저장과 Kafka Event 발행을 절대 하나도 빠짐없이 정확히 한 번만 처리해야 합니다. 어떻게 설계하시겠습니까?"

60~90초 안에:

- dual write
- outbox
- at-least-once
- consumer idempotency
- exactly-once의 한계

를 포함해 답변해보세요.

## 평가 기준

- dual-write failure를 정확히 설명하는가
- Outbox의 Local Transaction 역할을 이해하는가
- 중복 발행 가능성을 인정하고 Idempotency를 연결하는가
- CDC와 Outbox를 구분하는가
- exactly-once를 과장하지 않는가

## Evaluation

- 정확도: Pending
- 실무 연결: Pending
- 꼬리질문 대응: Pending
- 재시험 필요 여부: Pending
