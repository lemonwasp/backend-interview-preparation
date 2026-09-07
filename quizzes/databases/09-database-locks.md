# 09. Database Locks — Knowledge Check

## 기본 확인

1. Database Lock의 목적은 무엇인가?
2. Shared Lock과 Exclusive Lock의 차이는?
3. MVCC DB에서도 Lock이 필요한 이유는?
4. Row / Page / Table Lock의 Granularity 차이는?
5. `SELECT ... FOR UPDATE`는 언제 사용하는가?

## Pessimistic / Optimistic

6. Pessimistic Lock의 기본 가정은 무엇인가?
7. Optimistic Lock은 version column으로 어떻게 충돌을 감지하는가?
8. Optimistic Lock Update의 affected rows가 0이면 무엇을 의미할 수 있는가?
9. 충돌이 매우 잦은 Hot Row에 Optimistic Lock을 쓰면 어떤 문제가 생길 수 있는가?
10. Pessimistic Lock이 항상 더 좋은 선택이 아닌 이유는?

## Range와 Contention

11. Phantom 방지를 위해 Row Lock보다 넓은 Range/Predicate 보호가 필요할 수 있는 이유는?
12. Gap Lock / Next-Key Lock / Predicate Lock을 모든 DBMS에서 동일한 개념처럼 말하면 안 되는 이유는?
13. Hot Row가 Throughput을 제한하는 원리를 설명하라.
14. 적절한 Index가 Lock Contention을 줄이는 데 도움을 줄 수 있는 이유는?
15. Long Transaction이 Lock Wait를 키우는 이유는?

## Backend 연결

16. Lock Wait가 DB Connection Pool Exhaustion으로 번질 수 있는 흐름을 설명하라.
17. 외부 API 호출을 `SELECT FOR UPDATE`와 COMMIT 사이에 두는 것이 위험한 이유는?
18. 재고 차감에 Atomic UPDATE, Pessimistic Lock, Optimistic Lock 중 무엇을 선택할지 어떤 기준으로 판단할 것인가?
19. Row 접근 순서를 일관되게 유지하는 것이 왜 중요한가?
20. Database Lock을 60초 안에 설명하라.

## 평가 기준

- MVCC와 Lock의 관계를 설명할 수 있다.
- Pessimistic / Optimistic Lock을 정확히 구분한다.
- Lock Granularity와 Contention의 Trade-off를 이해한다.
- Lock Wait가 Backend resource exhaustion으로 전파되는 흐름을 설명한다.

상태: 답변 대기
