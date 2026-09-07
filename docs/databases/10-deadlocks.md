# 10. Database Deadlocks

## 한 줄 정의

Deadlock은 **두 개 이상의 Transaction이 서로가 가진 Lock을 기다리면서 아무도 앞으로 진행할 수 없는 순환 대기 상태**입니다.

쉽게 말하면:

> A는 B가 놓기를 기다리고, B는 A가 놓기를 기다린다.

---

## 가장 단순한 예시

Transaction A:

```text
1. Row 1 Lock 획득
2. Row 2 Lock 대기
```

Transaction B:

```text
1. Row 2 Lock 획득
2. Row 1 Lock 대기
```

결과:

```text
A waits for B
B waits for A
```

둘 다 스스로 빠져나올 수 없습니다.

---

## SQL 예시

Transaction A:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Row 1 lock

UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Row 2를 기다릴 수 있음
```

Transaction B:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 50 WHERE id = 2;
-- Row 2 lock

UPDATE accounts SET balance = balance + 50 WHERE id = 1;
-- Row 1을 기다릴 수 있음
```

A와 B의 접근 순서가 반대라면 Deadlock이 발생할 수 있습니다.

---

## Coffman Conditions

운영체제 Deadlock과 같은 네 조건으로 설명할 수 있습니다.

1. Mutual Exclusion
2. Hold and Wait
3. No Preemption
4. Circular Wait

Database에서도 핵심은 Circular Wait입니다.

---

## DB는 Deadlock을 어떻게 처리하는가?

일반적인 DBMS는 Deadlock을 감지하면 그대로 영원히 기다리지 않습니다.

보통:

1. Wait-for 관계를 추적
2. Cycle 감지
3. Transaction 하나를 Victim으로 선택
4. 해당 Transaction을 Rollback/Abort
5. 다른 Transaction이 진행

즉 Deadlock은 **DB가 자동으로 감지해서 한쪽을 실패시키는 정상적인 동시성 이벤트가 될 수 있습니다.**

Application은 이 실패를 처리해야 합니다.

---

## Deadlock vs Lock Wait Timeout

둘은 다릅니다.

### Deadlock

순환 대기입니다.

```text
A → B 기다림
B → A 기다림
```

진행 가능성이 없으므로 DB가 Cycle을 감지할 수 있습니다.

### Lock Wait Timeout

순환이 없어도 Lock을 너무 오래 기다리면 Timeout이 날 수 있습니다.

```text
A가 Lock을 오래 보유
B가 A를 기다림
```

A는 언젠가 끝날 수도 있지만 B의 대기 한도를 넘었습니다.

---

## 가장 중요한 예방책: 같은 순서로 접근

예를 들어 항상 작은 account id부터 Lock합니다.

```text
항상 id 1 → id 2
```

모든 Transaction이 동일한 Lock Ordering을 따르면 Circular Wait 가능성을 크게 줄일 수 있습니다.

### 좋지 않은 패턴

```text
Request A: 1 → 2
Request B: 2 → 1
```

### 개선

```text
둘 다 min(id) → max(id)
```

---

## Transaction을 짧게 유지

Lock을 오래 잡고 있을수록 다른 Transaction과 겹칠 시간 창이 커집니다.

특히 위험한 패턴:

```text
BEGIN
UPDATE / SELECT FOR UPDATE
사용자 입력 대기
외부 API 호출
긴 계산
COMMIT
```

Transaction 안에서는 DB 작업만 빠르게 끝내는 것이 좋습니다.

---

## Index와 Deadlock

잘못된 Index나 넓은 Scan은 필요 이상으로 많은 Row/Range에 Lock 영향을 줄 수 있습니다.

예를 들어 조건에 적합한 Index가 없어 넓은 범위를 Scan하면:

- Lock 범위 증가
- Lock 보유 수 증가
- 충돌 확률 증가

할 수 있습니다.

따라서 Deadlock 분석에는 SQL뿐 아니라 실행 계획과 Index도 봐야 합니다.

---

## Deadlock은 버그인가?

항상 "절대 발생하면 안 되는 버그"라고만 보면 안 됩니다.

높은 동시성에서 Lock을 사용하는 시스템에서는 Deadlock 가능성을 완전히 0으로 만드는 것이 어렵거나 비효율적일 수 있습니다.

더 현실적인 전략은:

- 발생 가능성을 줄이고
- DB가 감지하게 하고
- Application이 안전하게 Retry할 수 있게 하는 것

입니다.

---

## Retry 시 주의점

Deadlock Victim이 된 Transaction은 보통 다시 시도할 수 있습니다.

하지만 Retry 전에 확인해야 합니다.

- Transaction 전체가 Rollback되었는가?
- 요청이 Idempotent한가?
- 외부 Side Effect가 이미 발생했는가?
- Retry 횟수 제한이 있는가?
- Backoff/Jitter가 필요한가?

단순 무한 Retry는 장애를 키울 수 있습니다.

---

## 외부 API와 Deadlock Retry

예를 들어 Transaction 내부에서 결제 API까지 호출했다면 위험합니다.

```text
DB 변경
결제 API 성공
Deadlock으로 DB Transaction Rollback
Application Retry
결제 API 또 호출
```

중복 결제가 발생할 수 있습니다.

따라서 외부 Side Effect는 Local DB Transaction과 자동으로 Rollback되지 않는다는 점을 기억해야 합니다.

대안:

- Idempotency Key
- Outbox Pattern
- Saga/Compensation
- Transaction 경계 재설계

등을 검토합니다.

---

## Deadlock을 조사할 때 볼 것

1. 어떤 Transaction들이 Victim/Survivor였는가?
2. 각 Transaction이 어떤 SQL을 실행했는가?
3. 어떤 Row/Index/Range Lock을 보유했는가?
4. Lock 획득 순서가 반대였는가?
5. Transaction이 너무 길지 않았는가?
6. Index가 적절한가?
7. Retry가 폭주하고 있지 않은가?

DBMS의 Deadlock Graph / Deadlock Log / Lock Monitoring 기능을 활용합니다.

---

## Application에서의 처리 흐름

```text
BEGIN
  business SQL
COMMIT
```

Deadlock Exception 발생:

```text
1. Transaction Rollback 확인
2. 재시도 가능한 작업인지 확인
3. 제한된 횟수 Retry
4. 필요하면 Backoff/Jitter
5. 계속 실패하면 Error 반환 + Log/Metric
```

---

## Deadlock vs OS Deadlock

원리는 같습니다.

OS에서는 Thread/Process가 Mutex 같은 자원을 기다립니다.

DB에서는 Transaction이 Row/Range/Table Lock을 기다립니다.

공통 핵심:

> 서로 자원을 보유한 채 상대 자원을 기다리는 Circular Wait

---

## 흔한 오해

### 오해 1. Deadlock은 Timeout과 같다

아닙니다. Deadlock은 Cycle, Timeout은 오래 기다린 결과입니다.

### 오해 2. Deadlock이 발생하면 DB가 멈춘다

보통 DBMS가 Victim Transaction을 abort해 Cycle을 해소합니다.

### 오해 3. Retry만 넣으면 해결이다

Retry는 복구 전략일 뿐입니다. Lock Ordering, Transaction 길이, Index, Hotspot을 개선해야 합니다.

### 오해 4. Deadlock은 항상 코드가 완전히 잘못된 증거다

고동시성 환경에서는 발생 가능성이 존재합니다. 다만 빈도가 높다면 설계 개선 신호입니다.

---

## 면접용 60초 답변

> Database Deadlock은 여러 transaction이 서로가 보유한 lock을 기다리면서 circular wait에 빠진 상태입니다. 예를 들어 A가 row 1을 잡고 row 2를 기다리는데 B는 row 2를 잡고 row 1을 기다리면 둘 다 진행할 수 없습니다. 대부분의 DBMS는 deadlock을 감지하면 transaction 하나를 victim으로 abort해서 cycle을 끊습니다. Application은 이 오류를 잡아 transaction이 안전하게 retry 가능한지 판단해야 합니다. 예방하려면 row 접근 순서를 일관되게 하고, transaction을 짧게 유지하며, 적절한 index로 lock 범위를 줄여야 합니다. Deadlock은 lock wait timeout과 다르고, retry만 넣기보다 원인인 lock ordering과 contention을 함께 개선해야 합니다.
