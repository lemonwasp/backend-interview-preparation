# 06. Transaction Isolation Levels

## 한 줄 정의

Isolation Level은 **동시에 실행되는 Transaction들이 서로의 중간 상태와 변경 결과를 어느 정도까지 보게 할지 정하는 규칙**입니다.

쉽게 말하면, 여러 사람이 같은 장부를 동시에 수정할 때 **서로 얼마나 강하게 간섭을 막을지** 정하는 설정입니다.

---

## 왜 필요한가?

Transaction이 하나씩만 실행된다면 문제는 단순합니다. 하지만 실제 Backend에서는 수많은 요청이 동시에 같은 DB를 사용합니다.

예를 들어 재고가 1개 남았을 때 두 요청이 동시에 다음을 수행한다고 합시다.

1. 재고를 읽는다.
2. 재고가 1 이상인지 확인한다.
3. 주문을 생성한다.
4. 재고를 1 감소시킨다.

동시성을 아무 제어 없이 허용하면 둘 다 재고가 있다고 판단할 수 있습니다.

반대로 모든 Transaction을 완전히 한 줄로 세우면 정확성은 높지만 처리량과 응답시간이 나빠질 수 있습니다.

따라서 DB는 **정확성 vs 동시성**의 Trade-off를 Isolation Level로 조절합니다.

---

## SQL 표준의 대표 Isolation Level

낮은 수준에서 높은 수준 순서로 보면 보통 다음 네 단계로 설명합니다.

1. Read Uncommitted
2. Read Committed
3. Repeatable Read
4. Serializable

하지만 중요한 점이 있습니다.

> 이름이 같아도 PostgreSQL, MySQL/InnoDB, SQL Server 등 DBMS별 구현 방식과 실제 보장 범위가 다를 수 있습니다.

면접에서는 표준 개념을 먼저 설명하고, 필요하면 DBMS별 차이를 덧붙이는 것이 안전합니다.

---

## 1. Read Uncommitted

다른 Transaction이 아직 COMMIT하지 않은 데이터까지 읽을 수 있는 가장 약한 수준입니다.

### 예시

Transaction A:

```text
balance = 100
UPDATE balance = 0
아직 COMMIT 안 함
```

Transaction B가 이 0을 읽을 수 있다면 Dirty Read입니다.

그런데 A가 나중에 ROLLBACK하면 B가 읽은 값은 실제로 존재한 적 없는 값이 됩니다.

### 특징

- 동시성은 높음
- 일관성 보장은 약함
- Dirty Read 가능
- 실무 OLTP에서 흔히 기본값으로 쓰이는 수준은 아님

---

## 2. Read Committed

**COMMIT된 데이터만 읽는다**는 수준입니다.

따라서 Dirty Read는 막습니다.

하지만 같은 Transaction 안에서 같은 Row를 두 번 읽었는데 값이 달라질 수 있습니다.

### 예시

Transaction A:

```text
SELECT price FROM products WHERE id = 1; -- 1000
```

그 사이 Transaction B:

```text
UPDATE products SET price = 1200 WHERE id = 1;
COMMIT;
```

Transaction A가 다시 읽으면:

```text
SELECT price FROM products WHERE id = 1; -- 1200
```

같은 Transaction 안인데 값이 달라졌습니다.

이것이 Non-repeatable Read입니다.

### 특징

- Dirty Read 방지
- Non-repeatable Read 가능
- Phantom Read 가능할 수 있음
- PostgreSQL 기본 Isolation Level은 일반적으로 Read Committed

---

## 3. Repeatable Read

같은 Transaction 안에서 이미 읽은 데이터는 다시 읽어도 같은 값을 보도록 더 강하게 보장합니다.

즉 Non-repeatable Read를 방지하는 것이 핵심입니다.

하지만 Phantom Read 처리 방식은 DBMS마다 차이가 큽니다.

예를 들어:

```sql
SELECT * FROM orders WHERE amount >= 10000;
```

첫 번째 조회에는 5개 Row가 있었는데 다른 Transaction이 조건을 만족하는 새 Row를 INSERT하고 COMMIT한 뒤 다시 조회했을 때 6개가 보이면 Phantom Read입니다.

SQL 표준 관점에서는 Repeatable Read에서 Phantom이 가능할 수 있지만, 실제 DBMS는 MVCC나 Gap/Predicate Lock 등으로 더 강한 동작을 제공할 수 있습니다.

### 특징

- Dirty Read 방지
- Non-repeatable Read 방지
- Phantom 여부는 구현을 확인해야 함
- Snapshot 기반 구현에서는 읽기 일관성이 강해질 수 있음

---

## 4. Serializable

동시에 실행되더라도 결과가 **어떤 순서로 하나씩 실행된 것과 동등하도록** 보장하려는 가장 강한 Isolation Level입니다.

중요한 점은 반드시 실제로 한 줄로 실행된다는 뜻은 아닙니다.

DB는 Lock, Predicate Lock, Serializable Snapshot Isolation, 충돌 감지 등의 방법으로 직렬화 가능한 결과를 만들 수 있습니다.

### 특징

- 가장 강한 논리적 격리
- 동시성 충돌 시 대기나 Transaction abort/retry가 증가할 수 있음
- 정확성은 강하지만 비용도 큼

---

## 핵심 이상 현상 표

개념적으로 다음처럼 기억하면 좋습니다.

| Isolation | Dirty Read | Non-repeatable Read | Phantom Read |
|---|---|---|---|
| Read Uncommitted | 가능 | 가능 | 가능 |
| Read Committed | 방지 | 가능 | 가능 |
| Repeatable Read | 방지 | 방지 | 표준상 가능할 수 있음 |
| Serializable | 방지 | 방지 | 방지 |

단, 실제 DBMS 구현은 이 표보다 더 강한 보장을 제공할 수 있습니다.

---

## Isolation이 높으면 무조건 좋은가?

아닙니다.

Isolation이 강해질수록 다음 비용이 생길 수 있습니다.

- Lock 대기 증가
- Abort / Retry 증가
- 동시 처리량 감소
- 오래 열린 Transaction의 부담 증가
- MVCC Version 정리 지연

따라서 핵심 질문은 이것입니다.

> 이 업무 규칙에서 어떤 동시성 이상 현상까지 허용할 수 있는가?

예를 들어 통계 조회와 계좌 이체는 필요한 Isolation 수준이 다를 수 있습니다.

---

## Backend에서의 실제 판단

### 사례 1. 상품 상세 조회

약간 오래된 값을 읽어도 큰 문제가 없다면 아주 강한 Isolation이 필요하지 않을 수 있습니다.

### 사례 2. 재고 차감

단순 SELECT 후 UPDATE만으로는 Lost Update 위험이 있습니다.

대안은 다음과 같습니다.

- 조건부 UPDATE
- `SELECT ... FOR UPDATE`
- Optimistic Lock
- Atomic SQL

Isolation Level만 올린다고 모든 비즈니스 경쟁 조건이 자동으로 해결되는 것은 아닙니다.

### 사례 3. 송금

잔액 불변식과 중복 처리가 중요하므로 Transaction 경계, Lock/MVCC, Idempotency를 함께 설계해야 합니다.

---

## 흔한 오해

### 오해 1. Serializable이면 무조건 느리다

항상 그런 것은 아닙니다. Workload와 구현에 따라 다릅니다. 다만 충돌이 많은 환경에서는 대기나 abort/retry 비용이 커질 수 있습니다.

### 오해 2. Repeatable Read는 모든 DB에서 동일하다

아닙니다. MySQL/InnoDB와 PostgreSQL의 동작을 완전히 동일하게 가정하면 안 됩니다.

### 오해 3. Isolation Level만 높이면 Lost Update가 항상 사라진다

업데이트 패턴과 DBMS 구현에 따라 별도 Lock, Version Check, Atomic UPDATE가 필요할 수 있습니다.

### 오해 4. Serializable은 실제 Single Thread 실행이다

아닙니다. 결과가 직렬 실행과 동등하도록 보장하는 것이 핵심입니다.

---

## 면접용 60초 답변

> Isolation Level은 동시에 실행되는 Transaction들이 서로의 변경을 어느 정도까지 볼 수 있는지 정하는 규칙입니다. 일반적으로 Read Uncommitted, Read Committed, Repeatable Read, Serializable 순으로 격리가 강해집니다. 격리가 약하면 Dirty Read, Non-repeatable Read, Phantom Read 같은 이상 현상이 발생할 수 있고, 강하게 설정하면 정확성은 높아지지만 Lock 대기나 abort/retry 등 동시성 비용이 커질 수 있습니다. 또한 같은 Isolation Level 이름이라도 PostgreSQL, MySQL 같은 DBMS별 구현 차이가 있으므로 실제 보장 범위를 확인해야 합니다. 실무에서는 무조건 가장 강한 수준을 쓰기보다 업무 불변식과 성능 요구를 기준으로 선택합니다.
