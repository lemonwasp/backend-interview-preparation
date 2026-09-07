# 25. Database Review — 60초 답변 체크포인트

Database 신규 개념 학습을 일단 여기서 멈추고, 01~24를 **면접에서 실제로 꺼낼 수 있는 수준**으로 압축합니다.

문서를 읽었다고 완료가 아닙니다. 아래 질문을 자료 없이 60초 안에 설명할 수 있어야 합니다.

---

## 1. Relational Model / Keys

> 관계형 DB는 Row와 Column으로 표현되는 Relation을 중심으로 데이터를 저장하고, Key와 Constraint로 식별성과 관계의 무결성을 유지합니다. Primary Key는 Row를 유일하게 식별하는 제약이고 Foreign Key는 다른 Table의 Key를 참조해 Referential Integrity를 유지합니다. Primary Key와 Index는 종종 함께 구현되지만 동일한 개념은 아닙니다.

## 2. B-Tree Index

> B-Tree 계열 Index는 높은 branching factor로 tree height를 낮춰 적은 page access로 값을 찾게 합니다. Equality와 Range Search에 유리하지만 Index가 많아질수록 Insert/Update 비용과 storage가 증가합니다. Composite Index는 column 순서가 중요하고 optimizer는 selectivity와 cost에 따라 Index를 사용하지 않을 수도 있습니다.

## 3. Clustered / Non-clustered Index

> Clustered Index의 핵심은 실제 Row 저장 순서 또는 Row 저장 구조와 Index key가 밀접하게 연결된다는 점이고, Non-clustered Index는 별도 구조에서 Row locator나 clustered key를 통해 실제 Row를 찾습니다. 정확한 구현은 InnoDB, SQL Server 등 DBMS마다 다릅니다.

## 4. Query Execution / EXPLAIN

> Query Optimizer는 여러 Physical Plan 후보 중 estimated cost가 낮은 계획을 선택합니다. EXPLAIN에서는 Index 사용 여부만 보는 게 아니라 rows read, estimated vs actual rows, join algorithm, sort/hash spill, lookup 횟수를 봐야 합니다. 많은 Row가 필요하면 Full Scan이 Index lookup보다 더 합리적일 수도 있습니다.

## 5. Transaction / ACID

> Transaction은 여러 DB 작업을 하나의 논리 단위로 묶고 ACID는 Atomicity, Consistency, Isolation, Durability를 설명합니다. Local Transaction은 같은 DB 안의 변경을 원자적으로 처리하지만 외부 API나 Message Broker까지 자동으로 rollback할 수는 없습니다.

## 6. Isolation Levels

> Isolation Level은 동시에 실행되는 Transaction이 서로의 변경을 어느 정도까지 보게 할지를 정합니다. 높은 isolation은 anomaly를 더 줄이지만 blocking, abort, version 관리 같은 비용이 증가할 수 있습니다. 세부 보장은 DBMS 구현에 따라 차이가 있습니다.

## 7. Dirty / Non-repeatable / Phantom Read

> Dirty Read는 commit되지 않은 값을 읽는 것, Non-repeatable Read는 같은 Row를 다시 읽었을 때 값이 달라지는 것, Phantom Read는 같은 조건의 Row 집합이 달라지는 것입니다. 이 외에도 Lost Update와 Write Skew 같은 anomaly를 별도로 봐야 합니다.

## 8. MVCC

> MVCC는 Row의 여러 version과 snapshot visibility를 사용해 reader와 writer의 충돌을 줄이는 구현 메커니즘입니다. Isolation Level 자체와 동일한 개념은 아니며, long transaction은 오래된 version 정리를 방해할 수 있습니다. MVCC가 있어도 writer-writer conflict와 explicit lock은 남습니다.

## 9. Database Locks

> Lock은 동시 Transaction이 같은 데이터에 접근할 때 consistency를 보호합니다. Shared/Exclusive, Row/Page/Table/Range Lock 등이 있고 `SELECT FOR UPDATE` 같은 pessimistic locking도 있습니다. Lock 범위와 유지 시간이 커지면 contention과 pool exhaustion으로 이어질 수 있습니다.

## 10. Deadlock

> Deadlock은 Transaction들이 서로가 가진 lock을 기다리는 circular wait 상태입니다. DBMS는 보통 한 Transaction을 victim으로 abort해 cycle을 끊습니다. 예방에는 일관된 lock ordering, 짧은 transaction, 적절한 index와 작은 lock 범위가 중요합니다.

## 11. Normalization

> Normalization은 functional dependency를 기준으로 데이터 중복과 insert/update/delete anomaly를 줄이는 설계 원칙입니다. 하지만 읽기 비용 때문에 의도적인 denormalization을 할 수도 있습니다. 주문 당시 가격처럼 역사적 snapshot은 단순 중복이 아니라 독립된 사실일 수 있습니다.

## 12. Optimistic vs Pessimistic Lock

> Optimistic Lock은 충돌이 드물다고 보고 version check로 commit 시점에 conflict를 감지합니다. Pessimistic Lock은 먼저 lock을 잡아 conflict를 차단합니다. Hot row에서는 optimistic retry storm과 pessimistic lock wait/deadlock 중 어떤 비용이 더 큰지 판단해야 합니다.

## 13. Unique Constraint / Upsert

> Application에서 SELECT 후 INSERT로 존재 여부를 검사하는 것만으로는 concurrent request에 안전하지 않습니다. UNIQUE Constraint 같은 DB invariant가 최종 방어선이 되어야 하고, Upsert는 존재 여부 판단과 write를 DB의 concurrency control 안에서 처리하는 데 유용합니다.

## 14. Replication / Read Replica

> Replication은 같은 데이터를 다른 node로 복제해 availability와 read scale을 높입니다. Async Replica는 replication lag 때문에 stale read가 가능하므로 read-after-write consistency가 필요한 요청은 Primary routing 같은 정책이 필요합니다. Replica는 logical corruption도 복제하므로 backup을 대체하지 않습니다.

## 15. Partitioning / Sharding

> Partitioning은 보통 한 DB 시스템 안에서 데이터를 나누는 것이고 Sharding은 여러 독립 DB node에 데이터를 분산하는 방식입니다. Shard key는 data와 traffic을 균등하게 분산해야 하며, cross-shard join, aggregate, transaction, unique constraint, rebalancing 비용이 커집니다.

## 16. DB Connection Pool / Transaction Boundary

> Connection Pool은 connection 생성 비용을 줄이는 성능 최적화이면서 동시에 DB에 들어가는 concurrency를 제한하는 보호 장치입니다. Long transaction, lock wait, connection leak은 pool exhaustion을 일으킬 수 있고 pool size를 무작정 키우면 DB saturation을 악화시킬 수 있습니다. 외부 API 호출은 DB transaction 안에 오래 붙잡아두지 않는 것이 일반적으로 안전합니다.

## 17. ORM / N+1

> N+1은 부모 목록 1회 조회 후 각 Row마다 관계 데이터를 별도 Query로 읽어 총 N+1개의 Query가 발생하는 문제입니다. Fetch Join, batch loading, projection 등으로 줄일 수 있지만 무조건 join을 늘리면 cartesian explosion이 생길 수 있으므로 실제 SQL과 query count를 확인해야 합니다.

## 18. Schema Migration

> 무중단 Migration은 기존 App과 새 App이 동시에 동작할 수 있게 backward-compatible하게 진행해야 합니다. 흔한 방식은 Expand → Migrate → Contract로, 새 schema를 먼저 추가하고 data backfill과 application 전환이 끝난 뒤 오래된 schema를 제거합니다. 큰 table DDL과 backfill은 lock, I/O, replica lag를 유발할 수 있습니다.

## 19. Pagination / Large Data Access

> OFFSET Pagination은 깊은 page로 갈수록 앞의 Row를 건너뛰는 비용이 커지고 concurrent write 시 중복/누락이 생길 수 있습니다. Keyset Pagination은 마지막 sort key를 기준으로 다음 범위를 조회해 deep pagination에 유리하며 stable ordering과 tie-breaker가 중요합니다.

## 20. Cache Consistency

> Cache-aside에서는 miss 시 DB를 읽어 cache에 채우고 update 시 cache invalidation을 고려합니다. DB commit과 cache invalidation은 atomic하지 않으므로 stale cache 가능성을 인정해야 합니다. TTL, versioning, event/outbox 기반 invalidation, stampede 방지 등을 workload에 맞게 사용합니다.

## 21. Backup / PITR

> Replication은 availability와 read scale을 위한 것이고 Backup은 과거 상태 복구를 위한 것입니다. PITR은 Base Backup에 WAL/Transaction Log를 재생해 특정 시점까지 복구합니다. 운영에서는 RPO/RTO와 정기 Restore Test가 중요하며 Replica는 잘못된 DELETE도 따라가므로 Backup을 대체할 수 없습니다.

## 22. WAL / Checkpoint / Crash Recovery

> WAL은 Data Page보다 복구에 필요한 log를 먼저 durable하게 기록하는 원칙입니다. 그래서 commit 직후 crash가 나도 log를 재생해 변경을 복구할 수 있습니다. Checkpoint는 dirty page를 정리하고 recovery 시작점을 줄여 crash recovery 시간을 관리합니다.

## 23. Database Failure Scenarios

> DB 장애는 먼저 scope를 좁힌 뒤 pool wait, timeout, retry, active session, lock wait, slow query, CPU, I/O, replica lag를 함께 봅니다. 즉시 mitigation으로 traffic shed, expensive feature disable, routing 변경을 하고 이후 root cause와 missing guardrail을 분석합니다.

## 24. Transactional Outbox / CDC

> DB write와 Broker publish를 따로 실행하면 dual-write failure가 생깁니다. Transactional Outbox는 business data와 event intent를 같은 DB transaction에 저장하고 Relay나 CDC가 Broker로 전달합니다. 중복 발행 가능성이 있으므로 at-least-once와 consumer idempotency를 함께 설계합니다.

---

# 반드시 비교할 수 있어야 하는 12개

1. Primary Key vs Index
2. Clustered vs Non-clustered Index
3. Full Scan vs Index Scan
4. Read Committed vs Repeatable Read vs Serializable
5. MVCC vs Isolation Level
6. Optimistic vs Pessimistic Lock
7. Lock Wait vs Deadlock
8. Replication vs Backup
9. Partitioning vs Sharding
10. OFFSET vs Keyset Pagination
11. Replica vs Backup/PITR
12. Outbox vs 단순 Dual Write

---

# 장애 시나리오 8개

## A. API 느림 + DB CPU 낮음

Lock wait, pool exhaustion, storage latency를 확인합니다.

## B. Write 직후 조회가 예전 값

Read Replica lag 또는 stale cache를 확인합니다.

## C. 간헐적 Transaction abort

Deadlock, serialization failure, optimistic lock conflict를 구분합니다.

## D. 배포 후 Query 수 급증

ORM N+1과 lazy loading을 확인합니다.

## E. 깊은 Page에서 조회 급격히 느림

OFFSET pagination과 index/order 조건을 확인합니다.

## F. Failover 직후 Retry로 중복 결제 위험

Commit 결과 불확실성과 idempotency를 확인합니다.

## G. 잘못된 DELETE가 Replica에도 반영됨

Backup/PITR을 사용해야 합니다.

## H. DB 저장은 됐는데 Kafka Event 없음

Dual Write 문제이며 Transactional Outbox를 검토합니다.

---

# Database를 Completed로 올리는 최소 기준

1. Mock Interview 40문항 중 최소 32문항 핵심 개념 혼동 없이 답변
2. Index / Transaction / Isolation / MVCC / Lock / Replication / Sharding 비교 질문 통과
3. 장애 시나리오에서 `지표 → 원인 가설 → 즉시 완화 → 근본 대책` 순서로 답변
4. 틀린 답을 해당 문서에 반영
5. 최소 D+1 Re-test 수행

현재 상태는 **Prepared / Review**이며 아직 Completed가 아닙니다.
