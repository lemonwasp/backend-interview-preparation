# 16. System Design Review — 60-second Answers & Trade-off Guide

> 상태: Prepared  
> Quiz: Pending  
> Re-test: Pending

System Design 면접의 목적은 제품 이름을 많이 아는 것이 아니라 **요구사항을 구조화하고, 병목을 찾고, 선택의 대가를 설명하는 것**이다.

## 1. 면접 전체 진행 순서

다음 순서를 기본 골격으로 사용한다.

1. **요구사항 확인** — 핵심 기능, 사용자, 읽기/쓰기, 실시간성, 정합성
2. **규모 추정** — DAU, Peak RPS, payload, storage growth, bandwidth
3. **API / Data Model** — 핵심 entity와 access pattern
4. **High-level Architecture** — client → LB/API → service → cache/DB/queue
5. **병목 식별** — hot key, hot row, DB write, queue lag, dependency
6. **확장 전략** — cache, replica, partition, async, horizontal scaling
7. **정합성 / 중복 처리** — idempotency, ordering, consistency model
8. **장애 대응** — timeout, retry, circuit breaker, degradation, DR
9. **Observability** — SLI/SLO, metrics, logs, traces
10. **Trade-off 요약** — 왜 이 선택이 현재 요구사항에 맞는가

좋은 답변은 처음부터 Kafka, Redis, Kubernetes를 꺼내는 답변이 아니다.

---

# 2. 15개 핵심 주제 — 60초 답변

## 01 Requirements / Capacity Estimation

System Design은 기술 선택보다 요구사항 확인에서 시작한다. Functional requirement와 latency, availability, consistency 같은 non-functional requirement를 분리하고, 평균이 아니라 Peak RPS를 본다. Read/Write ratio, payload size, storage growth, bandwidth를 대략 계산하면 어떤 계층이 먼저 병목이 될지 판단할 수 있다. 모든 숫자를 정확히 맞히는 것이 목표가 아니라 설계 의사결정을 정량화하는 것이 목적이다.

## 02 Stateless Service

Stateless Service는 요청 처리를 특정 App Instance의 local durable state에 의존하지 않는 구조다. 세션이나 상태는 DB, distributed cache 같은 공유 저장소에 두어 어느 instance가 요청을 받아도 처리할 수 있게 한다. 이 구조는 load balancing, horizontal scaling, rolling deployment에 유리하다. Sticky Session은 단순한 대안이지만 특정 instance 의존성과 load imbalance를 만든다.

## 03 Cache Design

Cache는 반복되는 비싼 조회를 원본보다 빠른 계층에 저장해 latency와 backend load를 줄인다. 가장 흔한 패턴은 Cache-Aside이며 miss 시 DB를 읽어 cache에 채우고, write 시 DB 변경 후 invalidate한다. 하지만 stale data, invalidation failure, stampede, hot key가 생길 수 있다. 따라서 TTL, single-flight, jitter, key 설계와 consistency 요구를 같이 봐야 한다.

## 04 Message Queue

Message Queue는 producer와 consumer를 시간적으로 분리하고 spike를 buffer한다. 비동기 처리, retry, fan-out에 유리하지만 eventual consistency, duplicate delivery, ordering, backlog라는 비용이 생긴다. At-least-once delivery라면 consumer idempotency가 필요하고, retry에는 backoff와 DLQ가 필요하다. Queue depth만 보지 말고 oldest message age와 downstream capacity를 함께 본다.

## 05 Load Balancing / Horizontal Scaling

Load Balancer는 여러 backend instance로 traffic을 분산하고 health check로 unhealthy instance를 제외한다. Stateless App은 horizontal scaling하기 쉽지만 App만 늘린다고 전체 시스템이 확장되는 것은 아니다. DB pool, cache, external API가 다음 병목이 될 수 있다. 배포와 scale-in 때는 readiness와 connection draining으로 in-flight request 손실을 줄여야 한다.

## 06 Database Scaling / Read-Write Pattern

DB scaling은 read bottleneck인지 write bottleneck인지 먼저 구분한다. Read-heavy라면 index, cache, read replica가 효과적일 수 있지만 replica lag과 read-after-write 문제가 생긴다. Write-heavy라면 transaction 단축, batching, async processing, partitioning/sharding을 고려한다. 특히 `instance count × pool size`가 DB connection capacity를 넘지 않는지 확인한다.

## 07 Consistency / Availability

Consistency는 모든 데이터를 항상 최신으로 보이게 만드는 단일 설정이 아니다. operation별로 Strong, Eventual, Read-after-write 같은 수준을 선택한다. CAP는 network partition이 발생했을 때 consistency와 availability를 동시에 완벽히 지킬 수 없다는 뜻이다. 금융 잔액과 추천 피드는 같은 consistency 요구를 가질 필요가 없다.

## 08 Distributed Idempotency

분산 시스템에서는 timeout 뒤 retry가 발생해 같은 요청이 여러 번 실행될 수 있다. Idempotency Key와 DB UNIQUE constraint 또는 conditional insert로 같은 operation의 소유권을 atomic하게 확보하면 중복 side effect를 막을 수 있다. 같은 key에 다른 payload가 들어오는 것도 검증해야 한다. Idempotency는 distributed lock과 다르며, exactly-once를 마법처럼 보장하는 개념도 아니다.

## 09 Rate Limiting

Rate Limiting은 quota, fairness, abuse 방지, downstream protection을 위해 요청 속도를 제한한다. Token Bucket은 일정 속도로 token을 충전하면서 burst를 허용할 수 있다. 여러 instance에서 global limit을 만들려면 gateway나 distributed coordination이 필요하다. Rate limiting은 quota 정책이고 backpressure는 처리 능력 피드백, load shedding은 실제 과부하에서 작업을 버리는 전략이라는 차이가 있다.

## 10 Distributed Lock

Distributed Lock은 여러 process/instance가 같은 logical resource를 동시에 수정하지 못하게 조정한다. 그러나 우선 DB UNIQUE constraint, conditional update, optimistic concurrency로 해결 가능한지 본다. Lease/TTL 기반 lock은 stale owner 문제가 있어 중요한 side effect에는 fencing token이 필요할 수 있다. Lock 자체는 transaction이나 exactly-once를 보장하지 않는다.

## 11 Service Boundary / Ownership

Service Boundary는 기술이 아니라 business capability, data ownership, transaction boundary를 기준으로 나눈다. 각 데이터에는 source of truth가 있어야 하고 다른 서비스가 owner DB를 직접 수정하면 coupling이 커진다. 강한 transaction이 자주 필요한 데이터는 같은 boundary에 둘 이유가 있다. 작은 팀이나 불안정한 도메인에서는 microservice보다 modular monolith가 더 좋은 선택일 수 있다.

## 12 Observability / SLO

Observability는 metrics, logs, traces를 이용해 시스템 내부 상태를 추론하는 능력이다. 서비스는 RED(rate, errors, duration), 자원은 USE(utilization, saturation, errors) 관점으로 본다. SLI는 측정값, SLO는 내부 목표, SLA는 외부 계약이다. 평균 latency보다 p95/p99와 saturation을 보고, error budget으로 신뢰성과 개발 속도의 균형을 잡는다.

## 13 Graceful Degradation

Graceful Degradation은 일부 dependency가 느리거나 실패해도 핵심 기능은 유지하도록 optional 기능을 줄이는 전략이다. 추천 서비스가 죽었다면 기본 목록을 보여주고, cache가 stale해도 허용 가능한 화면이라면 오래된 값을 쓸 수 있다. Circuit breaker, stale cache, partial response, feature shedding을 사용해 critical path와 downstream을 보호한다.

## 14 Failure Mode Reasoning

장애는 단순한 down보다 slow/partial failure가 더 위험할 수 있다. 하나의 dependency가 느려지면 thread/connection pool이 고갈되고 retry가 폭증해 cascading failure가 될 수 있다. 따라서 `증상 → 지표 → 원인 가설 → 즉시 완화 → 정합성 확인 → 복구/재처리 → 재발 방지` 순으로 사고한다. Redundancy도 같은 failure domain에 있으면 실질적인 redundancy가 아니다.

## 15 Multi-region / Disaster Recovery

Multi-region은 region 장애, global latency, data residency 요구를 해결하지만 consistency와 운영 복잡도를 크게 높인다. Active-Passive는 단순하지만 failover 시간이 있고, Active-Active는 낮은 latency와 활용률의 장점 대신 multi-writer conflict와 split brain 위험이 있다. 설계는 RPO/RTO에서 역산하며 Backup만 있는 것이 아니라 restore, promotion, traffic switch, secrets/configuration, failback까지 drill로 검증해야 한다.

---

# 3. 핵심 Trade-off 비교표

| 선택 | 장점 | 주요 비용 / 위험 | 적합한 상황 |
|---|---|---|---|
| Vertical Scaling | 단순함 | 한계와 SPOF 가능성 | 초기/중간 규모 |
| Horizontal Scaling | 처리량/가용성 확장 | shared dependency 병목 | Stateless App |
| Local Cache | 매우 빠름 | instance별 불일치 | read-mostly, 약한 정합성 |
| Distributed Cache | 공유 가능 | network hop, cache 장애 | 여러 instance가 같은 cache 필요 |
| Sync API | 즉시 결과, 단순 reasoning | 강한 coupling, latency 전파 | 즉시 응답 필요 |
| Async Queue | spike 흡수, decoupling | eventual consistency, duplicate | background/event processing |
| Strong Consistency | 단순한 최신성 모델 | latency/availability 비용 | 잔액, 재고 확정 |
| Eventual Consistency | 확장성과 가용성 | stale/intermediate state | feed, analytics |
| Read Replica | read scale | replica lag | read-heavy |
| Sharding | write/storage scale | cross-shard 복잡도 | 단일 DB 한계 도달 |
| Optimistic Concurrency | 낮은 충돌에서 효율적 | conflict retry | 충돌이 드문 수정 |
| Distributed Lock | 직관적 coordination | lease/stale owner/운영 복잡도 | 단순 atomic DB 연산으로 부족할 때 |
| Active-Passive Region | 단순한 write ownership | failover 시간 | DR 중심 |
| Active-Active Region | latency/availability | conflict/split brain | 정말 필요한 global workload |
| Microservices | 독립 배포/ownership | network/consistency/운영 비용 | 명확한 boundary와 규모가 있을 때 |
| Modular Monolith | 단순한 transaction/운영 | 독립 scale 제한 | 작은 팀, 변화 많은 도메인 |

---

# 4. 자주 섞이는 개념 구분

## Rate Limit vs Backpressure vs Load Shedding
- Rate Limit: 정책적으로 얼마까지 허용할지
- Backpressure: consumer가 처리 가능한 속도를 producer에 전달
- Load Shedding: 이미 과부하일 때 일부 요청을 버려 핵심 기능 보호

## Idempotency vs Distributed Lock
- Idempotency: 같은 operation이 여러 번 실행되어도 결과 중복 방지
- Lock: 동시에 한 owner만 critical section을 수행하도록 coordination

## Replication vs Sharding
- Replication: 같은 데이터를 복제
- Sharding: 데이터를 나눔

## Cache vs Replica
- Cache: 원본이 아닌 빠른 임시 계층
- Replica: DB 원본 데이터의 복제본

## Availability vs Durability
- Availability: 지금 요청을 처리할 수 있는가
- Durability: commit된 데이터가 장애 후에도 남는가

## SLO vs SLA
- SLO: 내부 서비스 신뢰성 목표
- SLA: 고객과의 외부 계약/책임

## Failover vs Failback
- Failover: 장애 시 secondary로 전환
- Failback: 복구 후 원래 또는 정상 topology로 되돌림

---

# 5. 설계 면접 체크리스트

답변 중 다음 질문을 스스로 던진다.

- 핵심 user flow는 무엇인가?
- Peak RPS는 어느 정도인가?
- read/write ratio는?
- latency 목표는?
- 어떤 데이터는 stale해도 되는가?
- duplicate가 생기면 어떤 side effect가 위험한가?
- 가장 먼저 병목이 될 shared dependency는?
- cache가 죽으면 DB가 버틸 수 있는가?
- queue가 밀리면 무엇을 제한해야 하는가?
- DB timeout 뒤 commit 여부가 불명확하면 어떻게 하는가?
- 한 region이 통째로 죽으면 어떻게 하는가?
- 무엇을 metric으로 보고 어떤 SLO를 걸 것인가?

---

# 6. 장애 시나리오 8개

## A. Cache outage
Cache miss가 급증해 DB QPS와 connection wait가 상승한다. Cache 자체 복구만 보지 말고 DB 보호를 위해 rate limit, stale cache, partial degradation을 검토한다.

## B. Queue backlog
Queue depth와 oldest age가 증가한다. Consumer만 무작정 늘리면 DB가 죽을 수 있으므로 downstream saturation을 먼저 확인한다.

## C. DB write latency spike
Lock wait, hot row, disk latency, connection pool, slow query를 분리해 본다. Retry를 늘리기 전에 원인을 확인한다.

## D. Replica lag
사용자가 방금 쓴 데이터를 읽지 못한다. Read-after-write가 필요한 요청은 primary routing 또는 consistency-aware routing을 적용한다.

## E. Retry storm
Dependency latency가 상승했는데 모든 caller가 즉시 retry한다. timeout budget, exponential backoff+jitter, circuit breaker, concurrency limit가 필요하다.

## F. Duplicate payment request
첫 요청이 timeout됐지만 commit됐을 수 있다. Idempotency key와 atomic unique ownership으로 중복 결제를 막고 기존 결과를 반환한다.

## G. Stale distributed lock owner
Lease가 끝난 뒤 이전 owner가 늦게 write한다. Fencing token을 downstream이 검증해 오래된 owner의 write를 거부한다.

## H. Region failure
Traffic switch만으로 끝나지 않는다. DB promotion/replication state, queue, cache, object storage, secrets/config, DNS/TTL, RPO/RTO를 확인한다.

---

# 7. 면접용 한 문장 원칙

> 좋은 System Design은 가장 복잡한 구조가 아니라, 현재 요구사항을 만족하면서 실패 방식과 운영 비용까지 설명할 수 있는 최소한의 구조다.

## Status

- Explanation: Prepared
- Mock interview: Pending
- Re-test: Pending
- Completion: Not yet
