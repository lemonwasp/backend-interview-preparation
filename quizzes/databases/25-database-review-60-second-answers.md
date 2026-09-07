# Database Mock Interview — 40 Questions

Status: **답변 대기**

## Fundamentals

1. Primary Key와 Index의 차이를 설명하세요.
2. Foreign Key는 어떤 문제를 막나요?
3. B-Tree Index가 Equality와 Range Search에 적합한 이유는 무엇인가요?
4. Composite Index에서 column 순서가 중요한 이유는 무엇인가요?
5. Index가 있는데도 Full Scan을 선택할 수 있는 이유는 무엇인가요?
6. Clustered와 Non-clustered Index를 설명하세요.
7. Covering Index란 무엇인가요?
8. EXPLAIN에서 가장 먼저 보는 항목은 무엇인가요?

## Transactions / Concurrency

9. ACID를 각각 설명하세요.
10. Read Committed와 Repeatable Read의 차이를 설명하세요.
11. Dirty Read / Non-repeatable Read / Phantom Read를 구분하세요.
12. Lost Update란 무엇인가요?
13. Write Skew는 무엇인가요?
14. MVCC와 Isolation Level의 차이는 무엇인가요?
15. Long Transaction이 MVCC에 어떤 문제를 만들 수 있나요?
16. Shared Lock과 Exclusive Lock을 설명하세요.
17. `SELECT ... FOR UPDATE`는 언제 쓰며 어떤 비용이 있나요?
18. Optimistic Lock과 Pessimistic Lock을 비교하세요.
19. Deadlock과 Lock Wait Timeout의 차이는 무엇인가요?
20. Deadlock을 줄이는 대표 방법을 말해보세요.

## Data Modeling / Scale

21. 3NF를 직관적으로 설명하세요.
22. Denormalization을 선택할 수 있는 경우는 언제인가요?
23. `SELECT → INSERT`가 concurrency-safe하지 않은 이유는 무엇인가요?
24. Unique Constraint가 application validation보다 강한 최종 방어선이 되는 이유는 무엇인가요?
25. Read Replica에서 stale read가 발생하는 이유는 무엇인가요?
26. Read-after-write consistency를 어떻게 보완할 수 있나요?
27. Replication과 Sharding의 차이는 무엇인가요?
28. 좋은 Shard Key의 조건은 무엇인가요?
29. Cross-shard Transaction이 왜 어려운가요?
30. Replica가 Backup을 대체하지 못하는 이유는 무엇인가요?

## Backend Operations

31. DB Connection Pool을 무작정 키우면 왜 위험한가요?
32. ORM N+1 문제를 어떻게 발견하고 해결합니까?
33. 무중단 Schema Migration의 Expand → Migrate → Contract를 설명하세요.
34. OFFSET과 Keyset Pagination을 비교하세요.
35. Cache-Aside에서 stale cache가 생길 수 있는 이유는 무엇인가요?
36. Backup과 PITR의 차이를 설명하고 RPO/RTO를 정의하세요.
37. WAL과 Checkpoint가 Crash Recovery에 어떻게 사용되나요?
38. API p99가 올랐는데 DB CPU는 낮습니다. 어떤 지표를 확인하겠습니까?
39. DB write는 성공했는데 Kafka Event가 발행되지 않는 문제를 어떻게 막겠습니까?
40. Transactional Outbox를 사용해도 Consumer Idempotency가 필요한 이유는 무엇인가요?

---

# Scenario Round

## Scenario 1 — Connection Pool Exhaustion

- API latency 증가
- Connection acquire timeout 증가
- DB CPU 40%
- Lock wait 증가

`관찰 → 가설 → 즉시 완화 → 근본 대책` 순서로 답변하세요.

## Scenario 2 — Stale Read

- 주문 생성 성공
- 바로 조회하면 없음
- 수 초 후 보임

가능한 원인 2개와 각각의 해결 방향을 설명하세요.

## Scenario 3 — Duplicate Payment Risk

- DB Failover 시점에 Client가 timeout
- 실제 commit 여부 불명확
- Client가 재시도하려 함

어떻게 설계해야 하나요?

## Scenario 4 — Migration Incident

- 새 Column 추가 후 전체 write latency 상승
- Replica lag 증가

원인 후보와 안전한 migration 방법을 설명하세요.

## Scenario 5 — Dual Write Failure

- Order INSERT 성공
- Kafka publish 실패

Transactional Outbox를 포함해 복구 가능한 구조를 설명하세요.

---

# Pass Criteria

- 40문항 중 32개 이상 핵심 개념 혼동 없이 답변
- 비교 문제에서 개념의 역할과 trade-off를 함께 설명
- 장애 문제에서 지표와 원인 가설을 연결
- DBMS별 구현 차이가 있는 부분을 과도하게 일반화하지 않음
- Retry를 말할 때 Idempotency를 함께 고려

## Evaluation

- Score: Pending / 40
- Fundamentals: Pending
- Concurrency: Pending
- Scale: Pending
- Operations: Pending
- Scenario reasoning: Pending
- D+1 Re-test: Pending
