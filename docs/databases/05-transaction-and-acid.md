# 05. Transaction and ACID

## 한 줄 정의

Transaction은 여러 DB 작업을 하나의 논리적 작업 단위로 묶는 것이고, ACID는 그 Transaction이 신뢰성 있게 동작하기 위한 핵심 성질을 설명한다.

---

## 1. 왜 Transaction이 필요한가?

계좌 이체를 생각해보자.

```text
A 계좌 -10000
B 계좌 +10000
```

A에서 돈을 뺀 뒤 서버가 죽어서 B에 넣지 못하면 데이터가 깨진다.

우리가 원하는 것은:

```text
둘 다 성공
또는
둘 다 실패
```

이다.

이 논리적 단위를 Transaction으로 묶는다.

---

## 2. 기본 흐름

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 10000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 10000
WHERE id = 2;

COMMIT;
```

문제가 생기면:

```sql
ROLLBACK;
```

으로 Transaction 전의 일관된 상태로 되돌릴 수 있다.

---

## 3. ACID

### A — Atomicity

Transaction의 작업이 전부 적용되거나 전부 적용되지 않아야 한다.

```text
A 출금 성공 + B 입금 실패
```

같은 중간 상태를 최종 결과로 남기지 않는다.

### C — Consistency

Transaction 전후로 DB가 정의한 무결성 규칙을 만족해야 한다.

예:

- Primary Key 유일성
- Foreign Key
- CHECK Constraint
- Application이 유지해야 하는 업무 규칙

주의할 점은 DB가 업무 의미를 자동으로 모두 이해한다는 뜻이 아니다. Application과 Schema가 올바른 규칙을 함께 정의해야 한다.

### I — Isolation

여러 Transaction이 동시에 실행될 때 서로의 중간 상태 때문에 이상한 결과가 생기지 않도록 제어하는 성질이다.

하지만 `완전히 혼자 실행한 것처럼 보인다`는 강도는 Isolation Level에 따라 달라진다.

이 내용은 다음 주제에서 자세히 다룬다.

### D — Durability

COMMIT이 성공했다고 응답된 뒤에는 시스템 장애가 나더라도 결과가 보존되어야 한다는 성질이다.

DBMS는 보통 WAL, Redo Log, fsync 계열 메커니즘 등을 활용해 Durability를 구현한다.

구체적인 구현은 DBMS마다 다르다.

---

## 4. COMMIT은 단순 메모리 변수 변경이 아니다

Durability를 보장하려면 DB는 데이터 Page 자체를 즉시 전부 Storage에 쓰는 대신 로그를 먼저 안전하게 기록할 수 있다.

개념적으로:

```text
Transaction changes
      ↓
Write-Ahead Log / Redo Log
      ↓ durable
COMMIT success
      ↓
Data pages may be flushed later
```

이 방식은 매 Transaction마다 모든 Data Page를 즉시 쓰는 것보다 효율적일 수 있다.

---

## 5. WAL — Write-Ahead Logging

핵심 규칙은:

> 변경된 Data Page를 Storage에 쓰기 전에 복구에 필요한 Log를 먼저 기록한다.

장애가 발생하면 DB는 Log를 이용해 필요한 변경을 다시 적용하거나 복구할 수 있다.

이 개념은 OS의 Page Cache / fsync와도 연결된다.

---

## 6. Transaction이 너무 길면 어떤 문제가 생길까?

긴 Transaction은 다음 문제를 만들 수 있다.

- Lock을 오래 잡음
- 다른 Transaction의 대기 증가
- Deadlock 가능성 증가
- MVCC Version 정리 지연
- Connection 점유
- Replication / Log 처리 부담

따라서 Transaction 범위는 일반적으로 필요한 데이터 일관성을 만족하는 최소 범위로 유지하는 것이 좋다.

---

## 7. DB Transaction과 HTTP 요청

Backend에서 흔한 구조:

```text
HTTP Request
  ↓
Service
  ↓
BEGIN
  ↓
DB operations
  ↓
COMMIT
  ↓
HTTP Response
```

하지만 Transaction 안에서 외부 API 호출을 오래 기다리는 것은 위험할 수 있다.

예:

```text
BEGIN
DB update
외부 결제 API 5초 대기
DB update
COMMIT
```

그동안 DB Lock / Connection을 계속 점유할 수 있다.

분산 시스템에서는 DB Transaction 하나로 외부 시스템까지 Atomic하게 묶을 수 없으므로 Saga, Outbox, Idempotency 같은 별도 패턴이 필요할 수 있다.

---

## 8. Local Transaction ≠ Distributed Transaction

한 DB 안의 ACID Transaction은 강력하지만:

```text
DB A
DB B
Kafka
외부 Payment API
```

까지 자동으로 하나의 Atomic Transaction으로 만들어주지는 않는다.

2PC 같은 Distributed Transaction 방식도 있지만 복잡도와 Availability trade-off가 있다.

실무에서는:

- Idempotency
- Outbox Pattern
- Saga
- Retry
- Compensation

등을 조합하는 경우가 많다.

---

## 9. 실패했는데 실제로 COMMIT됐을 수도 있는 문제

Network가 끊겨 Client가 COMMIT 결과를 받지 못했다고 하자.

```text
Client → COMMIT
DB → commit 성공
DB → response 전송 중 network failure
Client → timeout
```

Client 입장에서는 `실패인지 성공인지 모르는 상태`가 된다.

이런 불확실성 때문에 결제/주문 같은 작업에서는 Idempotency Key가 중요하다.

Network Retry와 DB Transaction은 별개 계층의 문제다.

---

## 10. Auto-commit

많은 DB/Driver는 별도 Transaction을 시작하지 않으면 Statement 하나를 자체 Transaction으로 처리하는 Auto-commit Mode를 사용한다.

예:

```sql
UPDATE users SET name = 'Alice' WHERE id = 1;
```

이 Statement 자체가 Commit될 수 있다.

여러 Statement를 하나의 Atomic Unit으로 묶으려면 명시적 Transaction 범위가 필요하다.

---

## 11. C# / Backend 연결

개념적인 예:

```csharp
using var transaction = await dbContext.Database.BeginTransactionAsync();

try
{
    // 여러 DB 변경
    await dbContext.SaveChangesAsync();
    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

중요한 것은 문법보다:

- 어떤 업무 규칙을 Atomic하게 묶을 것인가?
- Transaction이 얼마나 오래 열려 있는가?
- 외부 I/O를 안에 넣고 있지 않은가?
- 실패 시 Retry가 안전한가?

를 판단하는 것이다.

---

## 12. 자주 하는 오해

### "ACID면 동시성 문제는 전부 없다"

아니다. Isolation Level에 따라 Dirty Read, Non-repeatable Read, Phantom 같은 현상이 달라진다.

### "COMMIT되면 Data Page가 즉시 전부 Disk에 써진다"

반드시 그런 구현은 아니다. WAL/Redo Log를 먼저 Durable하게 만든 뒤 Data Page는 나중에 Flush할 수 있다.

### "DB Transaction으로 외부 API까지 Rollback할 수 있다"

일반적인 Local DB Transaction으로는 불가능하다.

---

## 13. 60초 면접 답변

> Transaction은 여러 DB 작업을 하나의 논리적 단위로 묶어서 전부 성공하거나 전부 실패하도록 관리합니다. ACID에서 Atomicity는 all-or-nothing, Consistency는 무결성 규칙 유지, Isolation은 동시 Transaction 간 간섭 제어, Durability는 Commit 이후 장애가 나도 결과를 보존하는 성질입니다. DB는 보통 WAL이나 Redo Log를 먼저 Durable하게 기록해 복구 가능성을 만든 뒤 Data Page를 나중에 Flush할 수 있습니다. 실무에서는 Transaction을 너무 길게 잡으면 Lock과 Connection 점유가 늘기 때문에 범위를 최소화하고, 외부 API까지 Local Transaction으로 묶을 수 없다는 점도 중요합니다.
