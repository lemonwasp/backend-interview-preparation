# 13. Unique Constraint and Upsert

## 한 줄 설명

`UNIQUE` Constraint는 **중복되면 안 되는 비즈니스 규칙을 DB가 Atomic하게 지키도록 하는 장치**이고, Upsert는 **이미 있으면 Update, 없으면 Insert**를 하나의 DB 연산으로 처리하는 패턴입니다.

---

## 왜 Application Check만으로 부족한가

다음 코드를 생각해봅시다.

```text
1. SELECT email='a@example.com' 존재 여부 확인
2. 없으면 INSERT
```

두 요청이 동시에 실행되면:

```text
A: SELECT -> 없음
B: SELECT -> 없음
A: INSERT
B: INSERT
```

Application에서는 둘 다 "없다"고 봤습니다.

이것이 **Check-then-act Race**입니다.

따라서 정말 중복되면 안 되는 값에는 DB Constraint가 필요합니다.

```sql
CREATE UNIQUE INDEX ux_users_email
ON users(email);
```

이제 동시 INSERT 중 하나만 성공합니다.

---

## Constraint는 최종 방어선이다

대표적인 invariant:

- 사용자 이메일은 유일
- 결제 provider transaction id는 유일
- 주문 번호는 유일
- `(user_id, coupon_id)`는 한 번만 존재
- Idempotency Key는 한 Scope 안에서 유일

이런 규칙을 Application `if`만으로 지키면 여러 Instance와 동시 요청에서 깨질 수 있습니다.

> 비즈니스 불변식이 DB 한 곳에서 표현 가능하다면 Constraint로 내려보낼 수 있는지 먼저 확인한다.

---

## Composite Unique Constraint

```sql
UNIQUE (user_id, coupon_id)
```

이것은 Coupon 자체가 한 번만 존재한다는 뜻이 아니라:

```text
같은 User가 같은 Coupon을 두 번 사용하면 안 된다.
```

는 규칙을 표현합니다.

Constraint의 Column 선택 자체가 비즈니스 의미입니다.

---

## Upsert

개념적으로:

```text
없으면 INSERT
있으면 UPDATE 또는 아무것도 하지 않음
```

입니다.

DBMS마다 문법과 세부 동작은 다릅니다.

예를 들어 PostgreSQL 계열에서는 `INSERT ... ON CONFLICT ...` 형태가 대표적입니다.

```sql
INSERT INTO counters(key, value)
VALUES ('login', 1)
ON CONFLICT (key)
DO UPDATE SET value = counters.value + 1;
```

중요한 점은 Application에서 별도의 존재 확인을 하고 두 Query로 나누기보다 **DB가 conflict detection과 변경을 하나의 동시성 제어 범위에서 처리**하도록 하는 것입니다.

---

## DO NOTHING 패턴

중복 요청을 무시하고 최초 Insert만 인정하고 싶다면 개념적으로:

```text
INSERT if absent
else ignore
```

패턴을 사용할 수 있습니다.

이 방식은 Idempotency Record를 생성할 때도 유용할 수 있습니다.

하지만 "중복이면 무조건 성공으로 간주"하기 전에 같은 Key가 같은 Request 의미인지 확인해야 합니다.

---

## Idempotency Key와 연결

예:

```text
idempotency_records
- scope
- key
- request_hash
- status
- response

UNIQUE(scope, key)
```

동시에 같은 Key가 들어와도 DB Unique Constraint가 최초 처리자를 하나로 정할 수 있습니다.

그 다음:

- 같은 Key + 같은 Request -> 이전 결과 재사용
- 같은 Key + 다른 Request -> Conflict 처리

처럼 설계할 수 있습니다.

---

## Unique Constraint와 Index

많은 RDBMS에서 Unique Constraint를 보장하기 위해 Unique Index 계열 구조를 사용합니다.

하지만 개념적으로 구분해야 합니다.

- Constraint: 데이터 규칙
- Index: 접근/검사에 사용되는 물리적 구조

"Unique Index가 있으니 Constraint와 완전히 같은 개념"이라고 설명하는 것은 좋지 않습니다.

---

## NULL과 UNIQUE

`NULL`을 Unique Constraint가 어떻게 취급하는지는 DBMS와 기능에 따라 세부 차이가 있을 수 있습니다.

따라서 면접에서는:

> NULL uniqueness semantics는 DBMS별 차이를 확인해야 한다.

라고 말하는 편이 안전합니다.

---

## Upsert도 모든 Race를 해결하지 않는다

예를 들어:

```text
재고가 0보다 크면 감소
```

같은 조건부 invariant는 단순 Upsert만으로 충분하지 않을 수 있습니다.

Atomic Conditional Update:

```sql
UPDATE inventory
SET stock = stock - 1
WHERE product_id = 10
  AND stock > 0;
```

처럼 조건과 변경을 하나의 Statement로 표현하는 것이 더 적절할 수 있습니다.

---

## Error를 정상적인 경쟁 결과로 볼 수 있다

동시 INSERT에서 Unique Violation은 반드시 "시스템 장애"가 아닙니다.

경쟁 상황에서는:

```text
둘 중 하나 성공
다른 하나 Unique Conflict
```

가 정상적인 concurrency outcome일 수 있습니다.

Application은 이를:

- 409 Conflict
- 기존 Resource 반환
- Idempotent success

등 비즈니스 의미에 맞게 처리할 수 있습니다.

---

## 흔한 오해

### 오해 1. INSERT 전에 SELECT하면 중복을 막을 수 있다

동시 요청에서는 Race가 있습니다.

### 오해 2. 모든 중복 검증은 Application에서 해야 한다

DB Constraint로 표현 가능한 invariant는 DB가 최종 방어선이 되는 것이 강력합니다.

### 오해 3. Upsert면 모든 동시성 문제가 해결된다

아닙니다. 복잡한 invariant, 여러 Row/Resource 관계, 외부 Side Effect는 별도 Transaction/Lock/Idempotency 설계가 필요합니다.

---

## 면접 60초 답변

> Unique Constraint는 이메일, 주문 번호, 사용자별 쿠폰 사용처럼 중복되면 안 되는 invariant를 DB가 동시 요청에서도 Atomic하게 보장하도록 하는 장치입니다. Application에서 먼저 SELECT하고 없으면 INSERT하는 방식은 두 요청이 동시에 '없음'을 볼 수 있기 때문에 race-safe하지 않습니다. 그래서 DB Constraint를 최종 방어선으로 두고, 이미 존재하면 Update하거나 무시해야 하는 경우에는 DBMS가 제공하는 Upsert 계열 기능을 사용할 수 있습니다. 다만 Upsert가 모든 동시성 문제를 해결하는 것은 아니며 재고 차감처럼 조건부 invariant는 Atomic Conditional Update나 Lock이 필요할 수 있습니다.
