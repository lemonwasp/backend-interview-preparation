# 08. Distributed Idempotency

## 한 줄 정의

Distributed Idempotency는 **같은 요청이 네트워크 재시도·중복 전달 때문에 여러 번 도착해도 비즈니스 side effect가 한 번만 의도대로 반영되도록 만드는 설계**다.

## 왜 필요한가

분산 시스템에서는 timeout이 `실패`를 의미하지 않는다.

예:

```text
Client -> Payment API -> DB COMMIT
          |
          └─ response가 network에서 유실
```

Client는 timeout을 보고 재시도한다.

하지만 서버는 이미 결제를 완료했을 수 있다.

그 결과:

- 중복 결제
- 중복 주문
- 포인트 중복 지급
- 메시지 중복 처리

가 생길 수 있다.

---

## 1. Idempotent Operation

같은 요청을 여러 번 수행해도 최종 결과가 한 번 수행한 것과 같도록 만드는 성질이다.

예:

```text
SET status = 'PAID'
```

는 비교적 idempotent하게 만들기 쉽다.

반면:

```text
balance = balance - 100
```

을 같은 요청으로 두 번 수행하면 결과가 달라진다.

즉 HTTP method 이름만 보고 idempotency가 자동 보장되는 것은 아니다.

---

## 2. Idempotency Key

대표 패턴이다.

Client가 요청마다 고유 Key를 보낸다.

```http
POST /payments
Idempotency-Key: pay-7f2a...
```

Server는:

1. Key 확인
2. 처음이면 처리
3. 결과 저장
4. 같은 Key 재요청이면 기존 결과 반환

한다.

---

## 3. 가장 중요한 것은 Atomicity다

나쁜 구현:

```text
1. idempotency key 존재 확인
2. 없음
3. payment 수행
4. key 저장
```

동시에 두 요청이 들어오면 둘 다 2번을 통과할 수 있다.

따라서 key ownership과 business write 사이의 동시성 제어가 필요하다.

예:

- UNIQUE Constraint
- 같은 DB Transaction
- conditional insert
- compare-and-set

### 예시

```sql
INSERT INTO idempotency_requests(key, status)
VALUES (?, 'PROCESSING');
```

`key`에 UNIQUE Constraint가 있다면 한 요청만 성공하도록 만들 수 있다.

---

## 4. Key와 Request Payload를 묶어야 한다

같은 Idempotency Key로 서로 다른 요청 payload가 오면 어떻게 할 것인가?

예:

```text
key = abc
amount = 1000
```

이후:

```text
key = abc
amount = 9000
```

같은 key라고 이전 결과를 그대로 반환하면 위험하다.

그래서 다음을 저장할 수 있다.

- request hash
- resource id
- response
- status
- expiration

같은 key + 다른 payload는 409 같은 오류로 거절할 수 있다.

---

## 5. PROCESSING 상태가 필요한 이유

요청 처리 도중 장애가 날 수 있다.

```text
PROCESSING -> COMPLETED
          -> FAILED?
```

그러면 재요청이 왔을 때:

- 기존 요청이 아직 진행 중인지
- 완료됐는지
- 복구 가능한 실패인지

구분할 필요가 있다.

단, 상태 머신 설계는 비즈니스 특성에 따라 달라진다.

---

## 6. Idempotency Key 보관 기간

영원히 저장하면 저장 공간이 증가한다.

너무 빨리 지우면 늦은 retry가 중복 처리될 수 있다.

따라서:

- retry window
- business risk
- client behavior
- storage cost

을 기준으로 TTL/retention을 정한다.

---

## 7. Message Queue에서도 필요하다

At-least-once delivery에서는 같은 message가 다시 올 수 있다.

Consumer는 예를 들어:

```text
processed_message_id
```

를 저장하고 중복 처리를 막을 수 있다.

하지만 `message id 확인`과 `business update`가 원자적이지 않으면 race가 다시 생긴다.

가능하면 같은 transaction 안에서 처리한다.

---

## 8. Exactly-once에 대한 현실적인 태도

`Exactly once`라는 표현을 너무 쉽게 쓰면 안 된다.

분산 시스템 전체의 외부 side effect까지 정확히 한 번만 실행시키는 것은 어렵다.

현실적으로 자주 쓰는 접근은:

- at-least-once delivery
- idempotent processing
- unique constraint
- transactional outbox
- deduplication

을 조합해 **효과적으로 한 번 처리된 것처럼 보이게 만드는 것**이다.

---

## 9. Idempotency와 Lock은 다른 문제다

### Idempotency

같은 요청의 반복 실행으로 인한 중복 side effect를 막는다.

### Lock

동시에 여러 actor가 같은 resource를 변경할 때 충돌을 통제한다.

예:

- 같은 결제 요청이 retry됨 → idempotency
- 서로 다른 두 주문이 마지막 재고 1개를 경쟁 → concurrency control / lock / conditional update

둘을 혼동하면 안 된다.

---

## 10. 흔한 오해

### 오해 1: POST는 원래 non-idempotent이므로 어쩔 수 없다

아니다. application-level idempotency를 설계할 수 있다.

### 오해 2: UUID Key만 있으면 끝난다

아니다. atomic ownership, payload validation, retention, failure state가 중요하다.

### 오해 3: Retry를 없애면 된다

네트워크 오류와 failover에서 retry는 필요할 수 있다. 중요한 것은 안전한 retry다.

### 오해 4: Distributed Lock이 있으면 Idempotency Key가 필요 없다

해결하는 문제가 다르다.

---

## Backend 설계 예시

결제 API:

```text
Client
  ↓ Idempotency-Key
API
  ↓
Idempotency Table (UNIQUE key)
  ↓ same transaction
Payment Record
  ↓
Outbox Event
```

이 구조는:

- duplicate request
- ambiguous timeout
- message publish dual-write

문제를 함께 줄일 수 있다.

---

## 60초 면접 답변

> Distributed Idempotency는 네트워크 timeout이나 retry 때문에 같은 요청이 여러 번 들어와도 비즈니스 side effect가 중복되지 않도록 만드는 설계입니다. 대표적으로 Idempotency Key를 사용하고, DB의 UNIQUE Constraint나 conditional insert로 한 요청만 key ownership을 얻도록 합니다. 중요한 것은 key 확인과 business write 사이의 atomicity이며, 같은 key로 다른 payload가 오지 않도록 request hash도 검증할 수 있습니다. Queue의 at-least-once delivery에서도 consumer deduplication이 필요합니다. Idempotency는 같은 요청의 반복 문제이고, 서로 다른 요청 간의 경합을 막는 distributed lock과는 별개의 문제입니다.