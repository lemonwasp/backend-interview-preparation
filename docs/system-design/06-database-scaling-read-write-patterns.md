# 06. Database Scaling / Read-Write Patterns

## 한 줄 정의

Database Scaling은 **데이터 접근량이 커질 때 읽기·쓰기 패턴과 정합성 요구를 기준으로 DB 병목을 분리하고 확장하는 설계**다.

## 파인만식으로 생각하기

작은 가게에서는 계산대 하나로 충분하다.

하지만 손님이 많아지면 문제가 두 종류로 갈린다.

- 가격표를 **보기만 하는 사람**이 많다.
- 실제로 결제하고 재고를 **바꾸는 사람**이 많다.

DB도 같다.

읽기와 쓰기는 병목의 성격이 다르므로 먼저 패턴을 나눠야 한다.

---

## 1. 먼저 Read / Write Pattern을 측정한다

확인할 것:

- Read RPS / Write RPS
- Peak traffic
- Query latency p95 / p99
- Hot table / hot row
- 데이터 증가량
- Transaction 길이
- Connection 수
- Cache hit ratio
- Replica lag

`DB가 느리다`만으로는 설계를 결정할 수 없다.

---

## 2. Vertical Scaling

더 강한 DB 서버를 사용하는 방식이다.

예:

- CPU 증가
- RAM 증가
- 더 빠른 Storage
- IOPS 증가

### 장점

- 구조 변경이 적다.
- 운영 복잡도가 낮다.
- Transaction / Join 모델을 그대로 유지하기 쉽다.

### 한계

- 비용이 급격히 커질 수 있다.
- 물리적 상한이 있다.
- 단일 DB 의존성은 남는다.

따라서 Horizontal Scaling보다 먼저 검토할 가치가 있는 경우가 많다.

---

## 3. Read Replica

읽기가 병목이면 Primary의 변경을 Replica에 복제하고 일부 Read를 Replica로 보낼 수 있다.

```text
             ┌─> Read Replica
App -> Primary
             └─> Read Replica
```

### 얻는 것

- Read throughput 증가
- 분석/조회 workload 분리

### 생기는 문제

- Replication Lag
- stale read
- read-after-write inconsistency
- failover / topology 복잡도

사용자가 방금 수정한 프로필을 바로 다시 조회해야 한다면 Replica가 최신 상태가 아닐 수 있다.

따라서 `모든 GET은 Replica` 같은 단순 규칙보다 consistency requirement에 따른 routing이 필요하다.

---

## 4. Cache는 DB Scaling의 일부지만 DB를 대체하지 않는다

반복되는 읽기가 많다면 Cache가 가장 값싼 확장일 수 있다.

```text
App -> Cache -> DB
```

하지만 Cache는:

- stale data
- invalidation
- hot key
- stampede

문제를 추가한다.

즉 DB를 줄이는 대신 consistency 문제를 관리하게 된다.

---

## 5. Write Scaling은 더 어렵다

읽기는 복사본을 늘리기 쉽지만, 쓰기는 같은 데이터의 최종 상태를 합의해야 한다.

대표 선택:

- Transaction/Index 최적화
- Batch Write
- Async Queue
- Hot Row 제거
- Partitioning
- Sharding

### 예: Like Counter

매 요청마다 한 Row를 갱신하면 hot row가 된다.

```sql
UPDATE posts
SET like_count = like_count + 1
WHERE id = ?;
```

트래픽이 매우 크다면:

- 이벤트를 Queue에 적재
- shard별 counter
- 일정 주기로 aggregate

같은 구조를 고려할 수 있다.

단, 즉시 정확한 값이라는 요구사항을 완화하는 대가가 생긴다.

---

## 6. Partitioning vs Sharding

### Partitioning

보통 하나의 DB 시스템 안에서 데이터를 논리적으로 나눈다.

예:

- 날짜별
- tenant별

### Sharding

여러 독립 DB Node로 데이터를 분산한다.

```text
user_id % 4
  0 -> Shard A
  1 -> Shard B
  2 -> Shard C
  3 -> Shard D
```

### Sharding 비용

- Cross-shard Join
- Cross-shard Transaction
- Global Unique Constraint
- Rebalancing
- Hot Shard
- Routing logic

따라서 `트래픽이 많다 = 바로 Sharding`이 아니다.

---

## 7. Read/Write Split의 핵심 질문

면접에서는 다음을 확인해야 한다.

1. 읽기 비율이 얼마나 높은가?
2. stale read를 허용할 수 있는가?
3. read-after-write가 필요한가?
4. 쓰기 hotspot은 어디인가?
5. Transaction이 shard를 넘는가?
6. 데이터 증가가 storage 문제인가 throughput 문제인가?

---

## 8. Connection Pool도 Scaling 요소다

App instance를 10개로 늘렸고 각 instance가 DB Connection 100개를 열면 잠재적으로 1,000개다.

```text
10 instances × 100 connections = 1000 DB connections
```

App Horizontal Scaling이 DB에 압력을 그대로 전가할 수 있다.

따라서:

- pool size
- DB max connection
- query latency
- transaction length

을 함께 본다.

---

## 9. 흔한 오해

### 오해 1: Replica를 늘리면 Write도 빨라진다

아니다. 일반적인 Primary-Replica 구조에서 Replica는 주로 Read Scaling에 도움을 준다.

### 오해 2: Sharding은 무조건 확장성의 정답이다

아니다. 분산 transaction과 운영 복잡도를 크게 늘린다.

### 오해 3: DB를 키우기 전에 Microservice부터 나눈다

서비스 분리는 DB 병목을 자동 해결하지 않는다.

### 오해 4: Read Replica는 항상 최신 데이터를 준다

비동기 복제라면 lag가 존재할 수 있다.

---

## Backend 설계 연결

예를 들어 주문 시스템이라면:

- 상품 목록: cache / replica 허용 가능
- 주문 생성: primary + transaction
- 결제 상태: 강한 read-after-write 필요 가능
- 통계: async replica / warehouse 가능

같은 시스템 안에서도 데이터 종류에 따라 consistency와 routing 전략이 다를 수 있다.

---

## 60초 면접 답변

> Database Scaling은 먼저 읽기와 쓰기 패턴을 분리해서 보는 것이 중요합니다. 읽기 병목은 cache나 read replica로 분산할 수 있지만 replication lag 때문에 stale read와 read-after-write consistency를 고려해야 합니다. 쓰기 병목은 더 어렵고 transaction 최적화, batching, queue, partitioning, sharding 등을 검토합니다. Sharding은 throughput을 늘릴 수 있지만 cross-shard join, transaction, rebalancing 같은 운영 비용이 크기 때문에 마지막 단계의 선택에 가깝습니다. 또한 App을 horizontal scale하면 전체 DB connection 수가 함께 증가하므로 connection pool과 DB capacity를 같이 계산해야 합니다.