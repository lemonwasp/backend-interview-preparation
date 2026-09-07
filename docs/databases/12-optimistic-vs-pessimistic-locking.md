# 12. Optimistic vs Pessimistic Locking

## 한 줄 설명

- **Pessimistic Locking**: 충돌이 날 것이라고 보고 먼저 Lock을 잡는다.
- **Optimistic Locking**: 충돌이 드물다고 보고 일단 진행한 뒤 마지막에 Version을 비교한다.

둘 다 같은 문제를 푸는 방식입니다.

> 여러 사용자가 동시에 같은 데이터를 바꿀 때 누가 최종 값을 확정할 것인가?

---

## Pessimistic Locking

대표적인 형태:

```sql
SELECT *
FROM inventory
WHERE product_id = 10
FOR UPDATE;
```

Transaction이 끝날 때까지 다른 Writer가 해당 Row를 수정하지 못하게 막습니다.

### 장점

- 충돌을 초기에 차단
- 강한 직렬화가 필요한 흐름에 이해하기 쉬움
- 재고 차감, 잔액 이전처럼 경쟁이 높은 Row에서 유리할 수 있음

### 비용

- Lock Wait
- Deadlock
- 긴 Transaction에 매우 취약
- DB Connection을 오래 점유
- Hot Row에서 throughput 저하

---

## Optimistic Locking

대표적으로 `version` Column을 사용합니다.

```text
products
- id
- stock
- version
```

읽은 값:

```text
stock = 10
version = 7
```

수정:

```sql
UPDATE products
SET stock = 9,
    version = version + 1
WHERE id = 10
  AND version = 7;
```

영향받은 Row가 0개라면 누군가 먼저 Version을 바꾼 것입니다.

즉 충돌을 감지하고 Retry 또는 사용자에게 다시 시도하도록 합니다.

---

## 핵심 차이

| 항목 | Optimistic | Pessimistic |
|---|---|---|
| 충돌 가정 | 드물다 | 자주 발생할 수 있다 |
| 충돌 시점 | Commit/Update 시 감지 | 작업 전에 차단 |
| DB Lock 점유 | 적음 | 많음 |
| Retry | 중요 | 상대적으로 적음 |
| Deadlock 위험 | 낮음 | 더 높음 |
| Hot Row | 충돌 Retry 폭증 가능 | Lock Wait 폭증 가능 |

---

## 언제 Optimistic이 좋은가

- 읽기가 많고 쓰기 충돌은 드문 경우
- 긴 사용자 Think Time이 있는 Web UI
- Lock을 오래 유지하면 안 되는 경우
- Version 기반 Conflict 처리 UX를 만들 수 있는 경우

예:

사용자가 게시글 수정 화면을 3분 열어두었다고 합시다.

그 3분 동안 DB Row Lock을 잡는 것은 나쁜 설계입니다.

대신 Version을 가지고 있다가 저장 시 충돌을 검사할 수 있습니다.

---

## 언제 Pessimistic이 좋은가

- 동일 Row에 경쟁이 높음
- 충돌 후 Rollback/Retry 비용이 큼
- 짧은 Transaction 안에서 즉시 확정해야 함

예:

```text
남은 좌석 1개
동시에 100명이 예약
```

이런 상황에서는 단순한 Application check만으로는 부족합니다.

DB Constraint, Atomic Update, Lock 등을 함께 고려해야 합니다.

---

## C# / ORM 연결

Entity Framework 계열에서도 Concurrency Token 또는 Version Column 개념을 사용할 수 있습니다.

중요한 것은 ORM 문법이 아니라 생성되는 SQL의 의미입니다.

```text
WHERE id = ? AND version = ?
```

처럼 **읽었던 Version이 아직 같은가**를 DB가 Atomic하게 검사해야 합니다.

Application에서 먼저 Version을 SELECT한 뒤 별도 UPDATE를 하면 중간에 Race가 다시 생길 수 있습니다.

---

## Optimistic Lock도 Retry Storm이 날 수 있다

충돌이 드물다는 가정이 깨지면:

```text
100 clients
 -> same hot row
 -> 99 conflict
 -> retry
 -> 다시 충돌
```

처럼 Retry 부하가 커질 수 있습니다.

따라서 Hot Key에는:

- Queue
- Serialize
- Atomic Increment/Decrement
- Partitioning
- Pessimistic Lock

등 다른 전략이 더 적합할 수 있습니다.

---

## Pessimistic Lock과 외부 API

다음은 위험합니다.

```text
BEGIN
SELECT ... FOR UPDATE
외부 결제 API 호출 2초
UPDATE
COMMIT
```

외부 I/O 동안 Lock과 Connection을 계속 점유합니다.

가능하면 Transaction 내부에서는 DB 작업만 짧게 수행하고, 외부 Side Effect는 Idempotency/Outbox/Saga 같은 별도 설계와 연결해야 합니다.

---

## 흔한 오해

### 오해 1. Optimistic Lock은 Lock을 전혀 사용하지 않는다

DB 내부에서는 UPDATE 자체를 처리하기 위한 Lock/Concurrency Control이 여전히 존재할 수 있습니다.

"Application-level conflict strategy"의 차이라고 보는 편이 안전합니다.

### 오해 2. Pessimistic Lock이 항상 더 안전하다

정합성은 좋아질 수 있지만 긴 Lock, Deadlock, 낮은 throughput이라는 비용이 있습니다.

### 오해 3. Version Check는 Application에서 하면 된다

아닙니다.

비교와 UPDATE가 DB에서 하나의 Atomic 조건으로 실행되어야 Race를 막을 수 있습니다.

---

## 면접 60초 답변

> Pessimistic Locking은 충돌이 날 가능성이 높다고 보고 작업 전에 Row Lock 등을 잡아 다른 Transaction을 기다리게 하는 방식입니다. 반면 Optimistic Locking은 충돌이 드물다고 가정하고 Version Column 같은 값을 조건에 포함해 UPDATE 시점에 충돌을 감지합니다. Optimistic 방식은 Lock 점유가 적고 Web 환경에 잘 맞지만 Hot Row에서는 Retry가 폭증할 수 있습니다. Pessimistic 방식은 충돌을 직접 차단하지만 Lock Wait, Deadlock, Connection 점유 비용이 있습니다. 따라서 충돌 빈도, Transaction 길이, Retry 비용을 보고 선택해야 하며, 어떤 방식이든 DB에서 Atomic하게 충돌을 검사해야 합니다.
