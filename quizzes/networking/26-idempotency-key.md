# Quiz 26. Idempotency Key

Status: 답변 대기

## 핵심 확인

1. Idempotency의 의미를 자신의 말로 설명하세요.
2. 결제 요청에서 Timeout 후 Retry가 왜 중복 결제를 만들 수 있나요?
3. `POST`가 항상 Non-idempotent라고 단정하면 안 되는 이유는 무엇인가요?
4. Idempotency Key를 받은 서버의 기본 처리 흐름을 설명하세요.
5. 같은 Key를 가진 두 요청이 동시에 들어오면 어떤 Race Condition이 발생할 수 있나요?
6. DB Unique Constraint 또는 Atomic Insert가 왜 필요한가요?
7. Request Fingerprint를 함께 저장하는 이유는 무엇인가요?
8. 같은 Key인데 Payload가 다르면 어떻게 처리하는 것이 안전한가요?
9. Idempotency 결과를 전체 Response로 저장하는 방식과 Resource ID만 저장하는 방식의 trade-off는 무엇인가요?
10. TTL이 너무 짧을 때 생길 수 있는 문제는 무엇인가요?
11. Idempotency Key가 Exactly-once를 자동으로 보장하지 않는 이유는 무엇인가요?
12. DB 저장과 외부 API 호출이 함께 있는 작업은 왜 Idempotency Key만으로 부족할 수 있나요?

## 면접 꼬리 질문

13. 사용자가 주문 버튼을 3번 빠르게 눌렀습니다. 서버 설계를 설명하세요.
14. Payment API가 성공했지만 Client가 응답을 받지 못했습니다. 어떤 방식으로 안전하게 Retry하겠습니까?
15. Idempotency Key 저장소가 장애 나면 어떻게 하겠습니까?
16. Redis를 Idempotency Store로 쓸 때 무엇을 주의해야 하나요?
17. Idempotency Key를 Client가 생성하는 것과 Server가 생성하는 것의 차이는 무엇인가요?

## 60초 답변

18. "Idempotency Key가 무엇이고 왜 필요한가요?"에 60초 이내로 답하세요.
