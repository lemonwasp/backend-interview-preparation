# 10. Distributed Lock

## 한 줄 정의

Distributed Lock은 **여러 Process / Instance / Node가 같은 논리적 자원을 동시에 변경하려 할 때 일정 시점에 제한된 주체만 작업하도록 조정하는 메커니즘**이다.

## 먼저 중요한 질문

분산 Lock을 도입하기 전에 물어야 한다.

> 정말 Lock이 필요한가?

많은 경우 더 단순한 DB primitive로 해결할 수 있다.

예:

- UNIQUE Constraint
- atomic conditional update
- optimistic version check
- `SELECT ... FOR UPDATE`
- queue partitioning / single consumer

Distributed Lock은 네트워크와 장애를 포함하기 때문에 process-local `lock`보다 훨씬 어렵다.

---

## 1. Process-local Lock과 차이

C#의:

```csharp
lock (sync)
{
    // critical section
}
```

은 같은 Process 안의 Thread만 조정한다.

App Instance가 3개라면:

```text
Instance A -> local lock A
Instance B -> local lock B
Instance C -> local lock C
```

서로의 Lock을 모른다.

따라서 distributed race는 막지 못한다.

---

## 2. 어떤 상황에서 필요한가

예:

- 하나의 정산 Batch를 한 Worker만 실행
- 특정 Tenant의 migration을 한 번에 하나만 수행
- 같은 Resource에 대한 expensive rebuild 중복 방지
- scheduler leader 역할 선택

하지만 `마지막 재고 1개 차감` 같은 데이터 정합성은 DB conditional update나 transaction이 더 직접적인 해법일 수 있다.

---

## 3. 기본 구조

공유 Coordination Store에 lock ownership을 기록한다.

```text
Worker A ─┐
Worker B ─┼─> Coordination Store
Worker C ─┘
```

Lock 정보 예:

```text
resource = invoice:2026-09
owner = worker-A
expires_at = ...
```

한 Worker만 ownership을 획득하도록 atomic operation이 필요하다.

---

## 4. Lease / TTL

Owner가 죽었는데 Lock이 영원히 남으면 시스템이 멈춘다.

그래서 Distributed Lock은 흔히 TTL이 있는 Lease 형태를 사용한다.

```text
Acquire -> lease 30 sec
          ↓
       renew
```

하지만 TTL이 있으면 새 문제가 생긴다.

- GC pause
- network delay
- process freeze
- stop-the-world pause

때문에 기존 Owner는 자신이 아직 Lock을 가지고 있다고 생각하지만 실제 Lease는 만료됐을 수 있다.

---

## 5. 가장 위험한 상황: Stale Owner

시간 흐름:

```text
1. Worker A lock 획득
2. A가 긴 pause
3. lease 만료
4. Worker B가 lock 획득
5. A가 깨어나 작업 계속
```

이제 A와 B가 동시에 critical action을 수행할 수 있다.

단순 TTL Lock만 믿으면 데이터가 깨질 수 있다.

---

## 6. Fencing Token

이 문제를 줄이는 대표 개념이다.

Lock을 획득할 때 단조 증가하는 token을 발급한다.

```text
A -> token 41
lease 만료
B -> token 42
```

Downstream resource는 최신 token보다 작은 요청을 거절한다.

```text
if token < last_seen_token:
    reject
```

A가 늦게 돌아와 token 41로 쓰려 해도 거절할 수 있다.

핵심:

> Lock service만이 아니라 실제 side effect를 받는 resource가 fencing을 이해해야 한다.

---

## 7. Lock Acquisition의 Atomicity

잘못된 구현:

```text
1. lock 존재 확인
2. 없음
3. lock 생성
```

두 Worker가 동시에 통과할 수 있다.

따라서 shared store가 제공하는 atomic primitive가 필요하다.

예:

- DB UNIQUE Constraint / conditional insert
- compare-and-set
- consensus 기반 coordination primitive
- 적절한 atomic key creation

---

## 8. Lock은 Transaction이 아니다

Lock을 잡았다고 작업 전체가 원자적이 되는 것은 아니다.

예:

```text
lock 획득
DB update 성공
외부 API 실패
lock 해제
```

Distributed Lock은 **동시 실행을 조정**할 뿐:

- rollback
- exactly-once
- distributed transaction

을 자동 제공하지 않는다.

---

## 9. Lock과 Idempotency

둘은 함께 필요할 수 있다.

### Lock

동시에 여러 actor가 같은 resource를 조작하지 않도록 조정한다.

### Idempotency

같은 operation이 retry되어도 side effect가 중복되지 않도록 한다.

예:

Lock Owner가 timeout 후 retry하는 경우 idempotency가 여전히 중요하다.

---

## 10. Lock Granularity

너무 큰 Lock:

```text
lock = all-orders
```

- contention 증가
- throughput 감소

너무 작은 Lock:

- coordination overhead 증가
- 복잡도 증가

예:

```text
order:{orderId}
tenant:{tenantId}:settlement
```

처럼 실제 invariant 단위에 맞춘다.

---

## 11. Lock Ordering과 Deadlock

여러 Distributed Lock을 동시에 잡으면 local/DB lock과 마찬가지로 순환 대기가 생길 수 있다.

```text
A: lock X -> wants Y
B: lock Y -> wants X
```

가능하면:

- deterministic ordering
- 작은 lock set
- timeout
- bounded retry

를 사용한다.

---

## 12. Lock Service 장애

Coordination Store가 실패하면 정책이 필요하다.

### 안전성을 우선

Lock 확인이 안 되면 작업 중단.

예:

- 중복 정산이 치명적

### 가용성을 우선

일부 중복을 허용하고 idempotency/reconciliation으로 복구.

예:

- rebuild job 중복 실행이 비용 문제일 뿐 데이터 파괴는 없음

업무 invariant에 따라 선택한다.

---

## 13. Distributed Lock이 필요 없는 대표 사례

### 1) 중복 Email 가입

DB `UNIQUE(email)`이 더 직접적이다.

### 2) 재고 감소

```sql
UPDATE inventory
SET quantity = quantity - 1
WHERE id = ?
  AND quantity > 0;
```

영향 Row 수로 성공 여부를 판단하는 atomic conditional update가 더 단순할 수 있다.

### 3) 같은 Message 중복 처리

Idempotent Consumer / UNIQUE message ID가 더 적합할 수 있다.

---

## 14. 흔한 오해

### 오해 1: Redis에 key 하나 만들면 완벽한 Distributed Lock이다

아니다. TTL expiry, stale owner, failover, network partition, fencing까지 고려해야 한다.

### 오해 2: Distributed Lock은 데이터 정합성의 만능 해법이다

아니다. DB constraint나 atomic update가 더 안전하고 단순한 경우가 많다.

### 오해 3: Lock을 얻었으니 Exactly-once다

아니다. crash/retry/외부 side effect가 남는다.

### 오해 4: TTL만 있으면 Owner crash 문제가 완전히 해결된다

아니다. lease가 만료된 stale owner가 다시 실행될 수 있다.

---

## 설계 체크리스트

Distributed Lock을 사용한다면 최소한 답해야 한다.

1. Lock resource는 무엇인가?
2. 누가 owner인가?
3. 획득은 atomic한가?
4. Lease TTL은 얼마인가?
5. Renewal은 어떻게 하는가?
6. Owner가 pause되면 어떻게 되는가?
7. Fencing이 필요한가?
8. Lock Service 장애 시 정책은?
9. 작업 자체는 idempotent한가?
10. 더 단순한 DB primitive로 해결할 수 없는가?

---

## 60초 면접 답변

> Distributed Lock은 여러 서버 인스턴스가 같은 논리적 자원을 동시에 변경하지 않도록 조정하는 메커니즘입니다. 다만 먼저 DB UNIQUE Constraint, conditional update, optimistic locking처럼 더 단순한 방법으로 해결할 수 있는지 확인합니다. Distributed Lock을 쓰면 atomic acquisition뿐 아니라 owner crash를 위한 lease/TTL, lease가 만료된 뒤 예전 owner가 다시 실행되는 stale-owner 문제를 고려해야 합니다. 중요한 작업에서는 fencing token을 사용해 오래된 owner의 write를 downstream이 거절하도록 설계할 수 있습니다. 또한 Lock은 transaction이나 exactly-once를 제공하지 않기 때문에 retry와 idempotency는 별도로 설계해야 합니다.