# 15. Partitioning and Sharding

## 한 줄 설명

Partitioning과 Sharding은 모두 큰 데이터를 나누는 전략이지만 범위가 다릅니다.

- **Partitioning**: 보통 하나의 DB 시스템 안에서 Table을 여러 Partition으로 나눈다.
- **Sharding**: 데이터를 여러 독립 DB Node에 분산한다.

핵심은 단순히 데이터를 쪼개는 것이 아니라:

> 어떤 Key로 나눌 것인가, 그리고 나눈 뒤 요청을 어떻게 올바른 위치로 보낼 것인가?

입니다.

---

## 왜 나누는가

데이터와 트래픽이 커지면 단일 Table/Node에서:

- Index가 너무 커짐
- Maintenance 비용 증가
- Write throughput 한계
- Storage 한계
- 특정 Query가 너무 많은 데이터를 읽음

같은 문제가 생길 수 있습니다.

---

## Table Partitioning

예를 들어 주문 Table을 월별로 나눌 수 있습니다.

```text
orders_2026_01
orders_2026_02
orders_2026_03
...
```

논리적으로는 하나의 `orders`처럼 다루되 DB가 내부적으로 Partition을 관리할 수 있습니다.

대표 기준:

- Range: 날짜 범위
- List: Region / Category
- Hash: Key Hash

---

## Partition Pruning

Query가 Partition Key를 잘 사용하면 필요한 Partition만 읽을 수 있습니다.

예:

```sql
SELECT *
FROM orders
WHERE ordered_at >= '2026-09-01'
  AND ordered_at < '2026-10-01';
```

월별 Range Partition이라면 해당 월 Partition만 읽을 수 있습니다.

하지만 Query가 Partition Key와 맞지 않으면 여러 Partition을 모두 뒤질 수 있습니다.

즉:

> Partitioning은 Query Pattern과 Partition Key가 맞을 때 효과가 크다.

---

## Sharding

Sharding은 서로 다른 DB Node로 데이터를 나눕니다.

예:

```text
Shard A: user_id 0 ~ 999999
Shard B: user_id 1000000 ~ 1999999
Shard C: user_id 2000000 ~ ...
```

또는:

```text
hash(user_id) % N
```

같은 방식으로 Routing할 수 있습니다.

---

## Shard Key가 가장 중요하다

좋은 Shard Key는 보통:

- 요청이 어느 Shard로 갈지 쉽게 결정 가능
- 데이터와 트래픽이 고르게 분산
- 자주 함께 조회하는 데이터가 같은 Shard에 위치
- 시간이 지나도 한 곳으로 집중되지 않음

특성을 원합니다.

---

## 나쁜 Shard Key와 Hotspot

예를 들어 Write가 시간 순으로 증가하는데 Range Sharding을 잘못하면 최신 Write가 모두 마지막 Shard로 몰릴 수 있습니다.

```text
Shard A: old data
Shard B: old data
Shard C: all current writes <- HOT
```

또는 유명 사용자 한 명에게 트래픽이 집중되는 서비스에서 `user_id`만으로 Shard하면 특정 Shard가 Hotspot이 될 수 있습니다.

---

## Hash Sharding

Hash 기반 분산은 데이터가 비교적 고르게 퍼질 수 있습니다.

하지만 단순히:

```text
hash(key) % N
```

을 쓰면 Shard 수가 바뀔 때 많은 Key의 위치가 바뀔 수 있습니다.

이 때문에 Consistent Hashing이나 Virtual Node 같은 아이디어가 사용되기도 합니다.

다만 DB Sharding에서는 실제 제품/아키텍처에 따라 Directory 기반 Routing, Range Map 등 여러 전략이 존재합니다.

---

## Cross-shard Query의 비용

Sharding의 가장 큰 대가는 **로컬성이 깨지는 것**입니다.

한 Shard 안에서는 간단했던 Query가:

```text
Shard A query
Shard B query
Shard C query
-> Application/Gateway에서 merge
```

가 될 수 있습니다.

특히:

- Global ORDER BY
- Aggregate
- Join
- Unique Constraint
- Transaction

이 어려워집니다.

---

## Cross-shard Transaction

예:

```text
Account A -> Shard 1
Account B -> Shard 2
```

송금이 두 Shard에 걸치면 단일 DB Transaction처럼 처리하기 어렵습니다.

가능한 접근:

- Distributed Transaction
- Saga
- Outbox/Event-driven compensation
- 데이터 배치 자체를 바꿔 같은 Shard에 둘 수 있는지 검토

Sharding은 단순 성능 최적화가 아니라 **Transaction Boundary를 바꾸는 아키텍처 결정**입니다.

---

## Global Unique ID

Shard마다 Auto Increment를 사용하면 ID 충돌이 생길 수 있습니다.

대안:

- UUID
- Snowflake 계열 ID
- Shard prefix
- 중앙 ID allocator

각각 정렬성, 크기, 중앙 의존성 trade-off가 있습니다.

---

## Rebalancing

Shard A가 너무 커지면 데이터를 옮겨야 할 수 있습니다.

하지만 이동 중에도 서비스는 계속 요청을 받습니다.

고려할 것:

- Dual Read/Write 여부
- Routing Table 변경
- 데이터 복사 시점
- Cutover
- Consistency 검증
- Rollback

Sharding은 처음보다 **나중에 Shard 수를 변경할 때** 더 어렵습니다.

---

## Partitioning vs Sharding

| 항목 | Partitioning | Sharding |
|---|---|---|
| 범위 | 보통 한 DB 시스템 내부 | 여러 독립 DB Node |
| Routing | DB가 처리하는 경우 많음 | App/Proxy/Shard Router 필요 가능 |
| Transaction | 비교적 단순 | Cross-shard는 복잡 |
| 확장 | Storage/Query 관리 | Horizontal scale-out |
| 운영 복잡도 | 중간 | 높음 |

---

## Replication vs Sharding

둘도 자주 혼동합니다.

- Replication: 같은 데이터를 여러 Node에 복사
- Sharding: 서로 다른 데이터 부분을 여러 Node에 분산

실제 시스템에서는 함께 쓸 수 있습니다.

```text
Shard 1 Primary -> Replica
Shard 2 Primary -> Replica
Shard 3 Primary -> Replica
```

---

## Sharding은 너무 일찍 하면 안 된다

Sharding은:

- Cross-shard Query
- Deployment
- Migration
- Backup/Restore
- Failover
- Observability

를 모두 복잡하게 만듭니다.

따라서 먼저:

- Query/Index 최적화
- Cache
- Read Replica
- Vertical Scaling
- Table Partitioning

으로 해결 가능한지 확인하는 것이 일반적으로 합리적입니다.

---

## 흔한 오해

### 오해 1. 데이터가 많으면 무조건 Sharding해야 한다

아닙니다. 단일 강력한 RDBMS도 매우 큰 데이터를 처리할 수 있고 Index/Partitioning/Archive 전략이 먼저일 수 있습니다.

### 오해 2. Sharding하면 모든 Query가 빨라진다

Cross-shard Query는 오히려 느리고 복잡해질 수 있습니다.

### 오해 3. Replica와 Shard는 같다

아닙니다. Replica는 복사, Shard는 분할입니다.

---

## 면접 60초 답변

> Partitioning은 보통 하나의 DB 시스템 안에서 큰 Table을 Range, List, Hash 같은 기준으로 나누고 Partition Pruning을 통해 관리성과 Query 효율을 높이는 방식입니다. Sharding은 데이터를 여러 독립 DB Node에 분산해 Storage와 Write throughput을 수평 확장하는 방식입니다. Sharding에서 가장 중요한 것은 Shard Key인데, 데이터와 트래픽이 고르게 분산되고 자주 함께 접근하는 데이터가 같은 Shard에 있도록 설계해야 합니다. 대가는 Cross-shard Join, Aggregate, Transaction, Global Unique Constraint가 복잡해지는 것입니다. 그래서 Sharding은 가능한 한 늦게 도입하고 Index, Cache, Read Replica, Partitioning으로 해결 가능한지 먼저 확인하는 것이 좋습니다.
