# Quiz — 08. Distributed Idempotency

## Status

- Explanation: Prepared
- Quiz: Pending
- Re-test: Pending

## Recall Questions

1. Timeout이 곧 서버 처리 실패를 의미하지 않는 이유는 무엇인가?
2. Idempotency의 정의를 백엔드 side effect 관점에서 설명하라.
3. Idempotency Key는 어떤 문제를 해결하는가?
4. `존재 확인 → 처리 → key 저장` 순서가 race-safe하지 않은 이유는 무엇인가?
5. UNIQUE Constraint가 Idempotency Key 설계에서 중요한 이유는 무엇인가?
6. 같은 key로 다른 payload가 오면 왜 위험한가?
7. PROCESSING / COMPLETED 같은 상태가 필요한 이유는 무엇인가?
8. Idempotency Key retention/TTL을 정할 때 고려할 것은 무엇인가?
9. At-least-once Queue Consumer에서 deduplication이 필요한 이유는 무엇인가?
10. Exactly-once를 분산 시스템 전체에 쉽게 보장하기 어려운 이유는 무엇인가?
11. Idempotency와 Distributed Lock의 차이는 무엇인가?
12. Idempotency와 Transactional Outbox를 함께 쓰면 어떤 문제가 줄어드는가?

## Interview Drills

### Q1
결제 API에서 timeout 후 client가 재시도했다. 중복 결제를 막는 구조를 설명하라.

### Q2
Idempotency Key 테이블을 만들었지만 중복 결제가 발생했다. 어떤 atomicity 문제를 의심하겠는가?

### Q3
같은 Idempotency Key로 amount가 다른 요청이 들어왔다. 어떻게 처리하겠는가?

### Q4
Queue Consumer가 message id를 메모리 HashSet에 저장해서 중복을 막고 있다. 여러 instance나 restart 상황에서 왜 부족한가?

### Q5
마지막 재고 1개를 서로 다른 주문 두 개가 동시에 가져가려 한다. Idempotency만으로 해결 가능한가?

## 60-second Test

자료 없이 60초 안에 다음을 포함해 설명한다.

- ambiguous timeout
- Idempotency Key
- atomic ownership
- UNIQUE Constraint
- payload validation
- at-least-once
- Idempotency vs Lock
