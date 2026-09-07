# 14. Failure Mode Reasoning

## 한 줄 정의

Failure Mode Reasoning은 시스템 구성요소가 **어떻게 실패할 수 있고, 그 실패가 어디까지 전파되며, 어떤 신호로 감지하고 어떻게 제한·복구할지**를 설계 단계에서 미리 추론하는 방법이다.

---

## 1. 왜 필요한가

분산 시스템은 "정상 동작"보다 실패 방식이 더 다양하다.

예를 들어 DB 호출 하나만 봐도:

- 느려짐
- timeout
- connection refusal
- partial failure
- deadlock
- replica lag
- failover
- ambiguous commit result

처럼 여러 형태가 있다.

따라서 "DB가 죽으면요?" 수준이 아니라 **실패 유형과 전파 경로를 구체적으로 나눠야 한다.**

---

## 2. 기본 사고 순서

하나의 구성요소를 볼 때 다음 질문을 반복한다.

1. 어떻게 실패할 수 있는가?
2. 어떤 증상으로 보이는가?
3. 어디까지 영향이 전파되는가?
4. 자동으로 감지 가능한가?
5. 즉시 완화 방법은 무엇인가?
6. 데이터 정합성 문제는 남는가?
7. 복구 후 재처리가 필요한가?

---

## 3. Failure Domain

Failure Domain은 같은 장애에 함께 영향을 받는 범위다.

예:

- Process
- VM/Container
- Host
- Availability Zone
- Region
- External Provider

같은 AZ 안에 replica를 3개 두어도 AZ 전체 장애에는 모두 영향을 받을 수 있다.

그래서 redundancy는 **서로 다른 failure domain에 분산되어야 의미가 있다.**

---

## 4. Partial Failure

분산 시스템의 핵심 특징이다.

한 서비스는 살아 있고 다른 서비스만 죽을 수 있다.

```text
Order Service: healthy
Payment Service: timeout
Database: healthy
```

프로세스 하나짜리 프로그램처럼 전체가 한 번에 성공/실패하지 않는다.

이 때문에 timeout, retry, idempotency, circuit breaker가 중요해진다.

---

## 5. Latency도 Failure다

서비스가 응답은 하지만 30초 걸리는 것도 실질적인 장애일 수 있다.

느린 dependency는:

- request thread/Task 점유
- connection pool 점유
- queue 증가
- p99 증가
- upstream timeout
- retry 증가

를 만들어 장애를 전파한다.

그래서 failure reasoning에서 "down"뿐 아니라 **slow**를 반드시 포함한다.

---

## 6. Retry Storm

Dependency가 느려졌을 때 모든 upstream이 동시에 retry하면 원래보다 더 많은 트래픽이 몰린다.

```text
원래 1,000 RPS
↓ 실패
각 요청 3회 retry
↓
최대 수천 RPS 추가
```

이를 완화하려면:

- bounded retry
- exponential backoff
- jitter
- circuit breaker
- retry budget

등을 사용한다.

---

## 7. Queue Failure Mode

Queue는 buffer이지만 무한 저장소가 아니다.

볼 지표:

- Queue Depth
- Oldest Message Age
- Consumer Throughput
- Retry Count
- DLQ Size

Failure mode 예:

```text
consumer 느려짐
→ backlog 증가
→ message age 증가
→ storage 증가
→ recovery 후 massive replay
→ downstream 재과부하
```

복구 시에도 replay rate를 조절해야 한다.

---

## 8. Cache Failure Mode

Cache 장애 예:

- cache node down
- hot key
- mass expiration
- stale data
- invalidation failure

특히 Cache가 죽으면 모든 요청이 DB로 몰리는 **Cache Failure → DB Failure** 전파가 가능하다.

그래서 DB가 cache miss 전체를 감당할 수 없는 경우에는 load shedding이나 stale serving 전략이 필요하다.

---

## 9. DB Failure Mode

예:

- slow query
- lock contention
- connection pool exhaustion
- disk saturation
- replication lag
- failover
- logical corruption

중요한 점:

> timeout을 받았다고 transaction이 반드시 rollback된 것은 아니다.

네트워크 단절 시 server에서 commit됐지만 client가 결과를 못 받았을 수 있다.

그래서 write retry에는 idempotency가 필요하다.

---

## 10. Dependency Chain

```text
Client
→ Gateway
→ Service A
→ Service B
→ DB
```

각 단계 timeout이 모두 5초면 최악의 경우 request budget이 통제되지 않는다.

상위 deadline을 하위 dependency로 전파해야 한다.

예:

```text
전체 budget = 1s
Gateway 50ms
A 처리 100ms
B timeout 500ms
DB timeout 250ms
```

정확한 숫자는 workload에 따라 다르지만 핵심은 **하위 timeout이 상위 deadline보다 길면 안 된다는 것**이다.

---

## 11. Blast Radius

Blast Radius는 장애가 영향을 미치는 범위다.

줄이는 방법:

- tenant isolation
- shard isolation
- bulkhead
- separate queue
- per-service pool
- rate limit
- canary deployment
- feature flag

예를 들어 한 대형 고객의 batch 작업 때문에 모든 고객 DB connection이 고갈되지 않도록 tenant별 concurrency limit을 둘 수 있다.

---

## 12. Single Point of Failure

SPOF는 하나의 구성요소 실패가 전체 서비스 장애로 이어지는 지점이다.

예:

- 단일 DB
- 단일 Load Balancer
- 단일 Region
- 단일 credential issuer

하지만 모든 SPOF를 제거하는 것은 비용이 크다.

따라서 SLO와 비즈니스 중요도에 맞춰 redundancy를 설계한다.

---

## 13. Dependency가 동시에 실패할 수 있다

"Cache가 죽으면 DB로 fallback"만 생각하면 부족하다.

Cache 장애 때문에 DB가 과부하되어 함께 죽을 수 있다.

즉 failure는 독립적이지 않을 수 있다.

또한 공통 원인으로 여러 구성요소가 동시에 실패할 수 있다.

예:

- 같은 AZ
- 같은 DNS
- 같은 certificate
- 같은 cloud account quota
- 같은 deployment bug

---

## 14. Change Failure

실제 장애의 큰 원인은 변경이다.

- deploy
- config
- schema migration
- feature flag
- dependency upgrade

따라서 장애 분석에서 반드시 묻는다.

> "최근 무엇이 바뀌었는가?"

운영 설계에는:

- canary
- rollback
- backward compatibility
- feature flag
- migration safety

가 포함된다.

---

## 15. Failure Injection / Game Day

문서로만 장애 대응을 믿으면 실제로 동작하지 않을 수 있다.

안전한 환경에서:

- dependency timeout
- instance kill
- network latency
- queue backlog
- cache failure

등을 재현해 대응이 실제로 작동하는지 검증할 수 있다.

이를 Chaos Engineering의 작은 형태로 볼 수 있다.

---

## 16. 면접 답변 프레임

장애 시나리오 질문에는 다음 순서가 강하다.

```text
증상
→ 영향 범위
→ 핵심 지표
→ 원인 가설
→ 즉시 완화
→ 데이터 정합성 확인
→ 재처리/복구
→ 재발 방지
```

예:

> Queue backlog가 증가했다면 queue depth뿐 아니라 oldest age와 consumer throughput을 보고, consumer 자체 병목인지 downstream DB 병목인지 분리합니다. DB가 이미 포화라면 consumer 수를 늘리지 않고 intake를 제한하거나 low-priority traffic을 줄입니다. 복구 후에는 backlog를 한꺼번에 replay하지 않고 downstream capacity에 맞춰 drain합니다.

---

## 17. 흔한 오해

### 오해 1: redundancy가 있으면 장애가 없다

아니다. 공통 failure domain이면 함께 실패할 수 있다.

### 오해 2: timeout은 단순한 에러다

아니다. write 작업에서는 결과가 불명확한 ambiguous state일 수 있다.

### 오해 3: retry는 항상 reliability를 높인다

아니다. overload 시 retry storm을 만들 수 있다.

### 오해 4: Queue를 쓰면 spike 문제가 해결된다

Queue는 시간을 벌 뿐 downstream capacity가 부족하면 backlog가 계속 증가한다.

---

## 18. 60초 기술면접 답변

> Failure Mode Reasoning에서는 각 구성요소가 down, slow, partial failure로 어떻게 실패할 수 있는지 먼저 나누고, 그 영향이 어디까지 전파되는지 봅니다. 특히 latency 증가가 connection pool과 queue를 고갈시키거나 retry storm을 만드는 식의 cascading failure를 봐야 합니다. 장애 대응은 증상과 핵심 지표를 확인하고 원인 가설을 세운 뒤, circuit breaker, load shedding, bulkhead 같은 방법으로 blast radius를 줄입니다. Write timeout은 commit 여부가 불명확할 수 있으므로 idempotency와 reconciliation도 필요합니다. 복구 후에는 backlog replay나 데이터 정합성까지 확인하고 최근 변경 사항과 failure domain을 분석해 재발을 막습니다.
