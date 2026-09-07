# 09. Database Locks

## 한 줄 정의

Database Lock은 **동시에 접근하는 Transaction들이 서로 모순되는 변경을 하지 못하도록 특정 자원에 접근 규칙을 거는 동시성 제어 장치**입니다.

쉽게 말하면:

> 지금 내가 이 값을 안전하게 바꾸는 동안, 충돌하는 다른 작업은 잠깐 기다려라.

---

## 왜 필요한가?

MVCC가 읽기와 쓰기의 충돌을 많이 줄여주지만, 같은 Row를 두 Transaction이 동시에 수정하려는 상황까지 없애지는 못합니다.

예를 들어 두 요청이 같은 주문 상태를 동시에 바꾸려 한다고 합시다.

- A: `PAID → CANCELLED`
- B: `PAID → SHIPPED`

둘 다 자유롭게 쓰게 두면 최종 상태가 업무 규칙과 어긋날 수 있습니다.

Lock은 이런 충돌을 직렬화하거나 막는 데 사용됩니다.

---

## Shared Lock / Exclusive Lock

개념적으로 가장 기본적인 두 종류입니다.

### Shared Lock

여러 Reader가 동시에 공유할 수 있는 Lock입니다.

```text
Reader A: Shared
Reader B: Shared
```

서로 충돌하지 않습니다.

### Exclusive Lock

Writer가 데이터를 변경할 때 사용하는 배타적 Lock입니다.

```text
Writer A: Exclusive
```

충돌하는 다른 Shared/Exclusive 접근을 막을 수 있습니다.

단, 실제 MVCC DBMS에서는 일반 SELECT가 항상 전통적 Shared Lock을 잡는 것은 아닙니다. 읽기는 Snapshot Version으로 처리할 수 있습니다.

---

## Lock Granularity

Lock을 어느 크기의 자원에 거는지에 따라 비용과 동시성이 달라집니다.

대표적으로:

- Row Lock
- Page Lock
- Table Lock
- Predicate / Range Lock
- Metadata / Schema Lock

### 작은 단위

Row Lock처럼 범위가 작으면 동시성이 높습니다.

하지만:

- Lock 개수가 많아질 수 있음
- 관리 비용 증가

### 큰 단위

Table Lock처럼 범위가 크면 관리가 단순하지만 다른 Transaction을 많이 막습니다.

---

## Row Lock

가장 자주 접하는 형태입니다.

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 1;
```

해당 Row에 충돌하는 Update가 동시에 들어오면 한쪽이 기다릴 수 있습니다.

---

## `SELECT ... FOR UPDATE`

읽기만 하는 SELECT가 아니라, **곧 이 Row를 수정할 것이므로 미리 배타적 성격의 Lock을 확보하겠다**는 의도로 사용합니다.

예:

```sql
BEGIN;

SELECT stock
FROM products
WHERE id = 1
FOR UPDATE;

UPDATE products
SET stock = stock - 1
WHERE id = 1;

COMMIT;
```

### 장점

- 읽은 값과 이후 Update 사이의 경쟁을 줄일 수 있음

### 비용

- 다른 Transaction의 대기 증가
- Transaction이 길어질수록 Lock 보유시간 증가
- Deadlock 가능성 증가

---

## Pessimistic Lock

충돌 가능성이 높다고 가정하고 **먼저 Lock을 잡은 뒤 작업**하는 방식입니다.

대표 예:

```sql
SELECT ... FOR UPDATE
```

적합할 수 있는 상황:

- 동일 Row 충돌이 빈번함
- 실패 후 재시도 비용이 큼
- 강한 직렬화가 필요함

---

## Optimistic Lock

충돌이 드물다고 가정하고 Lock을 오래 잡지 않은 채 진행하다가 **마지막 Update 시 Version을 비교해 충돌을 감지**합니다.

```sql
UPDATE orders
SET status = 'SHIPPED',
    version = version + 1
WHERE id = 10
  AND version = 7;
```

Affected Rows = 0이면 누군가 먼저 수정했을 수 있습니다.

### 적합할 수 있는 상황

- 충돌이 드묾
- Read가 많음
- 재시도 가능

---

## Pessimistic vs Optimistic

| 항목 | Pessimistic | Optimistic |
|---|---|---|
| 기본 가정 | 충돌이 자주 난다 | 충돌이 드물다 |
| 전략 | 먼저 Lock | 마지막에 Version Check |
| 대기 | 발생 가능 | 적음 |
| 실패 처리 | 기다림 중심 | 충돌 시 Retry/Fail |
| 긴 Transaction | 특히 위험 | Version stale 가능 |

둘 중 하나가 무조건 우월한 것이 아닙니다.

---

## Gap / Range / Predicate Lock

Phantom 문제를 막으려면 기존 Row만 Lock해서는 부족할 수 있습니다.

예를 들어:

```sql
SELECT *
FROM bookings
WHERE seat_no BETWEEN 1 AND 10;
```

다른 Transaction이 그 범위 안에 새 Row를 INSERT하면 Row Lock만으로는 막지 못할 수 있습니다.

DBMS는 구현에 따라:

- Gap Lock
- Next-Key Lock
- Predicate Lock
- Serializable conflict tracking

등을 사용합니다.

이 역시 DBMS별 동작이 다르므로 일반화에 주의해야 합니다.

---

## Lock Contention

Lock 자체보다 실무에서 더 중요한 개념입니다.

여러 Transaction이 같은 Hot Row/Hot Range를 두고 경쟁하면:

- Lock Wait 증가
- Latency 증가
- Throughput 감소
- Timeout 증가
- Deadlock 가능성 증가

가 발생합니다.

### Hot Row 예시

```text
global_counter = global_counter + 1
```

모든 요청이 같은 Row를 수정하면 그 Row가 직렬화 지점이 됩니다.

---

## 긴 Transaction이 왜 위험한가?

Transaction이 길면 Lock을 오래 보유합니다.

예:

```text
BEGIN
SELECT ... FOR UPDATE
외부 API 5초 대기
UPDATE
COMMIT
```

5초 동안 다른 요청이 해당 Row를 기다릴 수 있습니다.

따라서:

> 외부 I/O를 DB Transaction 내부에서 오래 수행하지 않는다.

---

## Lock Escalation

일부 DBMS는 너무 많은 세부 Lock을 관리하는 비용을 줄이기 위해 더 큰 범위의 Lock으로 전환할 수 있습니다.

예:

```text
많은 Row Lock → Page/Table Lock
```

하지만 이 동작과 조건은 DBMS별로 다릅니다.

---

## DB Connection Pool과 Lock의 연결

Lock 대기가 길어지면 단순히 해당 Query만 느려지는 것이 아닙니다.

대기 중인 Transaction이 DB Connection을 계속 점유하면:

```text
Lock contention
→ Query latency 증가
→ Connection 반환 지연
→ Pool exhaustion
→ 전체 API 지연
```

으로 확산될 수 있습니다.

즉 Lock은 Database 내부 문제이면서 동시에 Backend 자원 고갈 문제입니다.

---

## Lock을 줄이는 설계

- Transaction을 짧게 유지
- 항상 같은 순서로 Row 접근
- 필요한 Row만 Lock
- 적절한 Index로 Lock 범위를 줄임
- Hot Row 설계 회피
- Atomic SQL 활용
- 충돌이 드물면 Optimistic Lock 검토
- 사용자 입력/외부 API를 Lock 보유 중 기다리지 않음

---

## 흔한 오해

### 오해 1. MVCC DB는 Lock을 쓰지 않는다

아닙니다. Writer-Writer 충돌, explicit lock, schema operation 등에서 Lock은 여전히 중요합니다.

### 오해 2. Row Lock이면 항상 딱 한 Row만 막힌다

Isolation Level, Index, 실행 계획, DBMS 구현에 따라 더 넓은 범위가 영향을 받을 수 있습니다.

### 오해 3. Pessimistic Lock이 항상 더 안전하다

정확성만이 아니라 contention, latency, deadlock 비용까지 고려해야 합니다.

### 오해 4. Optimistic Lock은 DB Lock이 전혀 없는 것이다

최종 UPDATE 자체는 DB 내부 동기화가 필요합니다. Optimistic이라는 말은 주로 Application 수준 충돌 전략을 뜻합니다.

---

## 면접용 60초 답변

> Database Lock은 동시에 실행되는 transaction들이 충돌하는 변경을 하지 못하도록 특정 row, range, table 같은 자원에 접근 규칙을 거는 동시성 제어 방식입니다. MVCC가 있어도 writer끼리 같은 row를 수정하면 충돌하기 때문에 lock은 여전히 필요합니다. `SELECT FOR UPDATE` 같은 pessimistic locking은 먼저 row를 잠그고 처리하며 충돌이 잦을 때 유용하지만 대기와 deadlock 비용이 있습니다. 반대로 optimistic locking은 version column을 이용해 마지막 update 시 충돌을 감지하고 retry합니다. 실무에서는 transaction을 짧게 유지하고, hot row와 불필요하게 넓은 lock 범위를 줄이며, lock wait가 connection pool exhaustion으로 번질 수 있다는 점까지 같이 봐야 합니다.
