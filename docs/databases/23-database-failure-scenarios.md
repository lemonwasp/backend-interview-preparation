# 23. Database Failure Scenarios

## 한 줄 요약

Database 장애 대응의 핵심은 **증상 → 관찰 지표 → 원인 가설 → 보호 조치 → 근본 원인** 순서로 접근하는 것입니다.

"DB가 느립니다"는 원인이 아닙니다.

---

# 1. Connection Pool Exhaustion

## 증상

- API latency 급증
- DB query 자체는 아주 느리지 않을 수도 있음
- Connection acquire timeout
- App Thread/Task 대기 증가

## 원인 후보

- Long Transaction
- Connection Leak
- Lock Wait
- Pool Size 부족
- App Instance 증가로 총 Connection 수 폭증
- DB 자체 saturation

## 확인할 것

- Pool active / idle / waiting
- Connection acquisition time
- Transaction duration
- DB active sessions
- Lock wait

## 잘못된 대응

> Pool Size부터 두 배로 늘린다.

DB capacity가 그대로면 전체 동시 Query만 늘어나 상황이 악화될 수 있습니다.

---

# 2. Lock Contention

## 증상

- 특정 Update API만 느림
- Query CPU는 낮을 수 있음
- Lock wait time 증가
- Transaction backlog

## 원인 후보

- Hot Row
- 너무 큰 Transaction
- 부적절한 Index로 Lock 범위 확대
- `SELECT ... FOR UPDATE` 남용

## 대응

- Transaction 짧게 유지
- Access/Lock ordering 정리
- Index 개선
- Optimistic Lock 고려
- Hot Key 분산

---

# 3. Deadlock

## 증상

- 특정 Transaction이 간헐적으로 abort
- Deadlock victim error

## 원인

A가 Row1 → Row2,
B가 Row2 → Row1 순서로 Lock을 잡는 식의 Circular Wait.

## 대응

- 일관된 Lock 순서
- Transaction 축소
- Index 개선
- Retry

단, Retry 전 Idempotency와 외부 Side Effect를 확인해야 합니다.

---

# 4. Slow Query / Bad Execution Plan

## 증상

- CPU 증가
- DB I/O 증가
- 특정 Query latency 급증
- Buffer cache miss 증가 가능

## 원인 후보

- Index 부재
- Cardinality Estimation 오류
- Statistics stale
- Parameter-sensitive plan
- 큰 Sort/Hash spill
- N+1
- OFFSET deep pagination

## 확인

- EXPLAIN / actual execution plan
- rows read vs returned
- estimated vs actual rows
- temp spill
- query count

---

# 5. Disk / Storage Saturation

## 증상

- 전체 Query latency 상승
- fsync/commit latency 상승
- Checkpoint 시 spike
- Queue depth 증가

## 원인 후보

- IOPS 부족
- Write burst
- Checkpoint
- Backup workload
- Large sort/temp files

Database CPU가 낮다고 해서 DB가 여유로운 것은 아닙니다.

---

# 6. Replica Lag

## 증상

- Write 직후 Read Replica에서 이전 값 조회
- Reporting 데이터 지연
- Lag metric 증가

## 원인 후보

- Primary write burst
- Replica CPU/I/O 부족
- Long query
- Network 문제

## 대응

- Read-after-write가 필요한 요청은 Primary로 routing
- Lag threshold 기반 Replica 제외
- Replica capacity 개선

---

# 7. Primary Failure / Failover

## 증상

- Connection reset
- 짧은 write outage
- DNS/endpoint 변경
- Transaction 중단

## Application이 고려할 것

- Connection 재수립
- Retry 가능 여부
- Idempotency
- Transaction 결과가 불확실한 경우

### 위험한 상황

Client는 timeout을 받았는데 실제 DB Transaction은 commit된 경우.

이때 무조건 다시 INSERT하면 중복 Side Effect가 날 수 있습니다.

---

# 8. Split-Brain

잘못된 Failover나 Network Partition 상황에서 두 Node가 모두 자신을 Primary라고 판단하면 충돌하는 Write가 발생할 수 있습니다.

고가용성 시스템은:

- Quorum
- fencing
- consensus/lease

같은 메커니즘으로 이를 방지하려 합니다.

---

# 9. Data Corruption / Logical Corruption

## Physical Corruption

- storage 문제
- page corruption

## Logical Corruption

- 잘못된 UPDATE/DELETE
- buggy migration
- application bug

Replica는 Logical Corruption을 그대로 복제할 수 있습니다.

따라서 Backup / PITR이 필요합니다.

---

# 10. Cache 때문에 DB 장애가 증폭되는 경우

DB가 느려져 Cache TTL 만료 후 대량 Miss가 발생하면:

```text
Cache Miss 폭증
  ↓
DB Query 폭증
  ↓
DB 더 느려짐
  ↓
Timeout/Retry 증가
```

이것이 장애 Amplification입니다.

필요한 보호:

- Cache Stampede 방지
- Rate Limit
- Backpressure
- Retry Budget
- Circuit Breaker

Networking Resilience 주제와 연결됩니다.

---

# 11. 장애 진단의 기본 순서

## Step 1. Scope

- 전체 API인가?
- 특정 Endpoint인가?
- Read인가 Write인가?

## Step 2. Application

- Pool wait
- Thread/Task queue
- timeout
- retry count

## Step 3. DB

- active sessions
- locks
- CPU
- I/O
- slow query
- transaction age
- replica lag

## Step 4. Recent Change

- deploy
- migration
- index change
- data volume spike

## Step 5. Mitigation

- traffic shed
- expensive feature disable
- replica routing
- rollback
- query kill

## Step 6. Root Cause

장애가 끝난 뒤:

- 왜 탐지가 늦었나
- 왜 blast radius가 컸나
- 어떤 guardrail이 없었나

까지 분석해야 합니다.

---

## 60초 면접 답변

> DB 장애는 먼저 범위를 좁힌 뒤 Application의 connection pool wait, timeout, retry와 DB의 active session, lock wait, slow query, CPU, I/O, replica lag를 같이 봅니다. 예를 들어 API latency가 높지만 DB CPU가 낮다면 lock contention이나 connection pool exhaustion일 수 있고, write 직후 stale read가 보이면 replica lag를 의심합니다. Failover에서는 timeout을 받은 Transaction이 실제로 commit됐을 가능성도 있으므로 retry 전에 idempotency를 확인해야 합니다. 장애 대응은 단순히 DB를 재시작하는 게 아니라 증상별 지표로 원인을 좁히고 traffic shedding이나 routing 같은 보호 조치로 blast radius를 줄이는 과정입니다.

---

다음: **24. Transactional Outbox / CDC**
