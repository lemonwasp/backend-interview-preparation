# 07. Dirty Read / Non-repeatable Read / Phantom Read

## 왜 따로 배워야 하나?

Isolation Level 이름만 외우면 실제 장애 상황에서 판단하기 어렵습니다. 먼저 **어떤 이상 현상이 발생하는지**를 이해해야 각 Isolation Level이 무엇을 막는지 연결할 수 있습니다.

---

## 1. Dirty Read

다른 Transaction이 아직 COMMIT하지 않은 값을 읽는 현상입니다.

### 예시

Transaction A:

```sql
UPDATE accounts SET balance = 0 WHERE id = 1;
-- 아직 COMMIT 안 함
```

Transaction B:

```sql
SELECT balance FROM accounts WHERE id = 1;
-- 0을 읽음
```

그 뒤 A가 ROLLBACK하면 B는 **결국 존재하지 않게 된 값**을 읽은 셈입니다.

### 핵심

> Dirty Read = Uncommitted Data를 읽음

---

## 2. Non-repeatable Read

같은 Transaction 안에서 **같은 Row를 두 번 읽었는데 값이 달라지는 현상**입니다.

Transaction A:

```sql
SELECT price FROM products WHERE id = 10; -- 1000
```

Transaction B:

```sql
UPDATE products SET price = 1200 WHERE id = 10;
COMMIT;
```

Transaction A:

```sql
SELECT price FROM products WHERE id = 10; -- 1200
```

### 핵심

> 같은 Row의 값이 바뀜

---

## 3. Phantom Read

같은 조건으로 범위 조회를 다시 했을 때 **조건을 만족하는 Row 집합 자체가 달라지는 현상**입니다.

Transaction A:

```sql
SELECT * FROM orders WHERE amount >= 10000;
-- 5 rows
```

Transaction B:

```sql
INSERT INTO orders(amount) VALUES (20000);
COMMIT;
```

Transaction A가 다시 같은 조건으로 조회했을 때 6 rows가 보이면 Phantom Read입니다.

### 핵심

> 같은 조건인데 Row 집합이 달라짐

---

## Non-repeatable vs Phantom

둘은 비슷해 보여도 기준이 다릅니다.

- Non-repeatable Read: **기존 Row의 값이 바뀜**
- Phantom Read: **조건에 맞는 Row의 존재/개수가 바뀜**

---

## 4. Lost Update

SQL 표준의 대표 세 이상 현상 외에도 실무에서 매우 중요한 문제가 Lost Update입니다.

두 Transaction이 같은 값을 읽고 각각 계산한 뒤 저장하면서 한쪽 변경이 사라지는 현상입니다.

초기 stock = 10

```text
A reads 10
B reads 10
A writes 9
B writes 9
```

두 번 감소했지만 최종값은 9입니다. 원래 기대는 8입니다.

### 해결 예시

Atomic UPDATE:

```sql
UPDATE products
SET stock = stock - 1
WHERE id = 1 AND stock > 0;
```

또는:

- `SELECT ... FOR UPDATE`
- Optimistic Lock with version
- 적절한 Isolation / Serializable 검토

---

## 5. Write Skew

MVCC/Snapshot 계열에서 자주 언급되는 더 미묘한 이상 현상입니다.

두 Transaction이 서로 다른 Row를 수정하지만, 둘이 함께 만족해야 하는 **전체 비즈니스 불변식**을 깨뜨릴 수 있습니다.

예를 들어 당직 의사가 최소 1명은 남아 있어야 하는데:

- A가 B가 당직인 것을 보고 자신을 off
- B가 A가 당직인 것을 보고 자신을 off

각 Transaction은 자신의 관점에서는 안전했지만 최종적으로 당직자가 0명이 될 수 있습니다.

이런 문제는 단순 Row-level 충돌만으로 잡히지 않을 수 있습니다.

---

## 6. Dirty Write

아직 COMMIT되지 않은 다른 Transaction의 변경을 또 덮어쓰는 현상입니다.

대부분의 실용적인 DBMS는 매우 낮은 Isolation에서도 Dirty Write를 막습니다. 하지만 개념적으로는 Read 이상 현상과 별도로 알아두면 좋습니다.

---

## 이상 현상을 왜 구분해야 하나?

같은 "동시성 문제"라도 해결 전략이 달라지기 때문입니다.

| 문제 | 대표 대응 |
|---|---|
| Dirty Read | Read Committed 이상 |
| Non-repeatable Read | Repeatable Read / Snapshot 계열 |
| Phantom Read | Serializable / Predicate·Range protection 등 |
| Lost Update | Atomic UPDATE, Lock, Version Check |
| Write Skew | Serializable 수준 또는 불변식 보호 설계 |

---

## Backend 사례

### 재고

`SELECT stock` 후 계산해서 저장하는 패턴은 Lost Update에 취약할 수 있습니다.

### 좋아요 수 / 조회수

Application에서 값을 읽어 `+1` 후 저장하기보다 DB Atomic Increment가 안전할 수 있습니다.

### 좌석 예약

"빈 좌석인지 읽기 → 예약 INSERT" 사이에 경쟁이 생길 수 있으므로 UNIQUE Constraint, Transaction, Lock 등을 함께 사용합니다.

### 계좌 잔액

단순 Isolation 용어보다 **잔액이 음수가 되지 않는 불변식**을 어떤 SQL/Constraint/Lock으로 지킬지가 중요합니다.

---

## 중요한 면접 포인트

### DB Constraint는 마지막 방어선이 될 수 있다

예를 들어 중복 예약을 막기 위해 Application에서 먼저 존재 여부를 조회해도 Race가 있습니다.

```text
A: 존재 안 함 확인
B: 존재 안 함 확인
A: INSERT
B: INSERT
```

UNIQUE Constraint가 있으면 DB 레벨에서 중복 상태 자체를 거부할 수 있습니다.

즉:

> Check-then-act만 믿지 말고, 가능한 불변식은 DB Constraint로도 표현한다.

---

## 흔한 오해

### 오해 1. Phantom Read는 같은 Row 값이 바뀌는 것이다

그건 Non-repeatable Read에 가깝습니다. Phantom은 조건에 맞는 Row 집합 변화입니다.

### 오해 2. Transaction이면 Race Condition이 없다

아닙니다. Isolation과 쿼리 패턴에 따라 경쟁 조건이 남습니다.

### 오해 3. `SELECT`로 확인한 뒤 `INSERT`하면 안전하다

동시에 다른 요청도 같은 확인을 할 수 있습니다. UNIQUE Constraint나 Lock이 필요할 수 있습니다.

---

## 면접용 60초 답변

> 대표적인 Transaction 이상 현상으로 Dirty Read, Non-repeatable Read, Phantom Read가 있습니다. Dirty Read는 아직 commit되지 않은 값을 읽는 것이고, Non-repeatable Read는 같은 transaction에서 같은 row를 다시 읽었을 때 값이 바뀌는 현상입니다. Phantom Read는 같은 조건의 범위 조회에서 row 집합 자체가 달라지는 현상입니다. 실무에서는 Lost Update도 중요합니다. 두 요청이 같은 값을 읽고 각각 수정하면서 한쪽 update가 사라질 수 있기 때문에 atomic update, row lock, optimistic version check 같은 방법을 사용합니다. 또한 check-then-insert 경쟁은 unique constraint 같은 DB 불변식으로 막는 것이 중요합니다.
