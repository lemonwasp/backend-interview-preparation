# Idempotency, Deduplication, and the Exactly-Once Illusion

## 1. 한 문장으로 설명

**Idempotency는 같은 요청이 여러 번 와도 최종 결과가 한 번 처리한 것과 같도록 만드는 성질이고, Deduplication은 중복 요청/메시지를 식별해 실제 처리를 한 번만 하도록 막는 기법이다. 분산 시스템에서는 네트워크 실패 때문에 완전한 exactly-once를 보장하기 어렵기 때문에 보통 at-least-once 전달 + idempotent 처리로 실질적인 exactly-once 효과를 만든다.**

---

## 2. 왜 필요한가?

결제 요청을 생각해보자.

```text
Client -> Payment API -> DB
```

클라이언트가 결제를 보냈고 서버는 실제 결제를 완료했지만, 응답 직전에 네트워크가 끊겼다고 하자.

```text
Client
  |
  | POST /payments
  v
Server ----> Payment completed
  |
  X response lost
```

클라이언트 입장에서는 성공했는지 알 수 없다.

그래서 재시도한다.

```text
POST /payments  (retry)
```

서버가 아무 보호 없이 다시 처리하면 이중 결제가 발생할 수 있다.

즉 분산 시스템에서는 **"요청이 실패했다"와 "요청의 응답을 받지 못했다"가 서로 다른 문제**다.

---

## 3. Idempotency란?

수학적으로는 대략 다음 성질이다.

```text
f(f(x)) = f(x)
```

같은 연산을 여러 번 적용해도 결과가 한 번 적용한 것과 같다는 뜻이다.

예:

```text
사용자 상태 = ACTIVE
```

을 여러 번 설정하는 것은 보통 안전하다.

```text
SET status = ACTIVE
SET status = ACTIVE
SET status = ACTIVE
```

반면 다음 연산은 기본적으로 idempotent하지 않다.

```text
balance -= 10,000
```

두 번 실행되면 결과가 달라진다.

---

## 4. HTTP 메서드와 Idempotency

일반적인 의미에서:

```text
GET     -> idempotent
PUT     -> idempotent
DELETE  -> idempotent한 의미로 설계 가능
POST    -> 기본적으로 idempotent하지 않음
```

예를 들어:

```http
PUT /users/42
{
  "name": "Alice"
}
```

를 여러 번 보내도 최종 상태는 동일할 수 있다.

하지만:

```http
POST /payments
```

는 호출할 때마다 새로운 결제를 만들 수 있기 때문에 별도 보호가 필요하다.

---

## 5. Idempotency Key

결제 API에서 자주 사용하는 방식이다.

클라이언트가 요청마다 고유 키를 만든다.

```http
POST /payments
Idempotency-Key: 8d2a1c...
```

서버는 이 키를 저장한다.

```text
idempotency_key | status    | result
----------------+-----------+-------------
8d2a1c...       | completed | payment_123
```

같은 키가 다시 오면 새 결제를 만들지 않고 기존 결과를 반환한다.

```text
1st request
key=A -> process -> save result

retry
key=A -> existing result -> return same response
```

핵심은 **클라이언트 재시도를 안전하게 만드는 것**이다.

---

## 6. 단순 조회 후 INSERT가 위험한 이유

다음 코드는 race condition이 있다.

```text
if key not exists:
    process payment
    insert key
```

동시에 두 요청이 들어오면:

```text
Request A: key 없음 확인
Request B: key 없음 확인
Request A: 결제 실행
Request B: 결제 실행
```

중복 결제가 발생할 수 있다.

그래서 DB 수준에서 unique constraint를 사용하는 것이 중요하다.

```sql
CREATE UNIQUE INDEX ux_idempotency_key
ON idempotency_requests(idempotency_key);
```

이렇게 하면 최종적으로 중복 키를 DB가 막아준다.

---

## 7. 처리 순서 설계

대표적인 구조는 다음과 같다.

```text
1. idempotency key 수신
2. key를 원자적으로 선점
3. 이미 존재하면 기존 상태/결과 확인
4. 비즈니스 로직 수행
5. 결과 저장
6. 같은 key 재요청 시 동일 결과 반환
```

중요한 점은 **key 저장과 실제 비즈니스 처리 사이의 실패 구간**을 고려해야 한다는 것이다.

예:

```text
key = processing 저장
        |
        v
payment 실행
        |
        X 서버 crash
```

이 경우 재시도 시 `processing` 상태를 어떻게 해석할지 정책이 필요하다.

---

## 8. Deduplication

Deduplication은 중복 입력을 찾아 실제 처리를 막는 기법이다.

메시지 큐 예시:

```text
Producer -> Broker -> Consumer
```

브로커가 같은 메시지를 두 번 전달할 수 있다.

```text
message_id = 123
message_id = 123
```

Consumer는 이미 처리한 message_id를 저장할 수 있다.

```sql
INSERT INTO processed_messages(message_id)
VALUES ('123');
```

`message_id`에 unique constraint를 걸어두면 같은 메시지의 두 번째 처리를 막을 수 있다.

---

## 9. At-most-once / At-least-once / Exactly-once

### At-most-once

```text
0번 또는 1번 전달
```

중복은 없을 수 있지만 손실 가능성이 있다.

### At-least-once

```text
1번 이상 전달
```

손실을 줄이는 대신 중복 전달이 가능하다.

그래서 consumer는 idempotent해야 한다.

### Exactly-once

```text
정확히 한 번
```

이상적으로는 가장 좋아 보이지만 네트워크와 장애가 있는 분산 시스템에서는 매우 비싸거나 특정 범위 내에서만 가능하다.

실무에서는 자주 다음 방식으로 해결한다.

```text
At-least-once delivery
        +
Idempotent consumer
        +
Deduplication
        =
Exactly-once effect
```

---

## 10. 왜 Exactly-once가 어려운가?

Consumer가 메시지를 처리한 뒤 ACK를 보내기 직전에 죽는 상황을 보자.

```text
Broker -> Consumer
             |
             v
          DB update
             |
             X crash before ACK
```

Broker는 ACK를 받지 못했기 때문에 메시지를 다시 보낸다.

하지만 DB 업데이트는 이미 끝났다.

```text
redelivery -> duplicate side effect
```

즉 브로커는 consumer 내부에서 실제 처리가 완료되었는지 완벽하게 알 수 없다.

이 불확실성 때문에 중복 전달을 전제로 설계하는 경우가 많다.

---

## 11. Idempotent Consumer

메시지 큐 consumer에서는 보통 다음 패턴을 사용할 수 있다.

```text
BEGIN TRANSACTION

1. processed_messages에 message_id INSERT
2. 비즈니스 데이터 업데이트
3. COMMIT

ACK
```

둘을 같은 DB transaction 안에 넣으면:

```text
message_id 기록
+
business update
```

가 함께 성공하거나 함께 실패한다.

그 후 ACK를 보낸다.

메시지가 재전달되어도 `message_id` unique constraint가 중복 처리를 막는다.

---

## 12. Transactional Outbox와의 연결

다음 문제가 있다.

```text
DB update success
Message publish failure
```

또는 반대로:

```text
Message publish success
DB update failure
```

이를 해결하기 위한 대표 패턴이 Transactional Outbox다.

```text
DB Transaction
  |
  +-- business data update
  +-- outbox row insert
```

둘을 한 transaction으로 묶고, 별도 publisher가 outbox를 읽어 메시지를 전송한다.

이때 publisher도 재시도할 수 있으므로 consumer 쪽 idempotency와 deduplication이 다시 중요해진다.

---

## 13. Idempotency Key 저장 시 고려할 것

### TTL

idempotency key를 영원히 저장하면 데이터가 무한히 커진다.

업무 특성에 맞춰 TTL을 둘 수 있다.

```text
24 hours
7 days
30 days
```

### Request fingerprint

같은 key인데 payload가 다르면 위험하다.

예:

```text
key = abc
amount = 10,000
```

이후:

```text
key = abc
amount = 1,000,000
```

따라서 key와 함께 request hash를 저장하고 동일 key에 다른 payload가 들어오면 거절할 수 있다.

### Response caching

첫 번째 요청의 status code와 response body를 저장해 같은 요청에 동일 결과를 돌려줄 수 있다.

---

## 14. Idempotency가 만능은 아니다

다음 작업은 외부 side effect 때문에 더 어렵다.

```text
- 이메일 발송
- SMS 발송
- 외부 결제 API 호출
- 배송 요청
```

내 DB에서 deduplication을 해도 외부 API가 이미 실행된 뒤 장애가 나면 상태가 불확실할 수 있다.

그래서 외부 시스템도 idempotency key를 지원하는지 확인하거나, reconciliation 작업을 두는 것이 중요하다.

---

## 15. Retry와 Idempotency의 관계

Retry는 실패 복구에 중요하지만 idempotency가 없으면 위험할 수 있다.

```text
Timeout
  |
  v
Retry
  |
  v
Duplicate side effect
```

따라서 일반적으로:

```text
Retry 가능
   |
   v
Idempotency 필요
```

특히 돈, 재고, 주문처럼 side effect가 있는 작업은 필수적으로 고려해야 한다.

---

## 16. 면접에서 자주 나오는 질문

### Q1. Idempotency란 무엇인가요?

> 같은 요청을 여러 번 수행하더라도 최종 결과가 한 번 수행한 것과 같도록 만드는 성질입니다. 분산 시스템에서는 timeout 이후 재시도가 흔하기 때문에 결제나 주문 같은 side effect를 안전하게 처리하기 위해 중요합니다.

### Q2. POST 요청을 어떻게 idempotent하게 만들 수 있나요?

> 클라이언트가 idempotency key를 보내고 서버가 해당 키와 처리 결과를 저장합니다. 같은 키가 다시 오면 새 작업을 수행하지 않고 기존 결과를 반환합니다. 동시에 같은 키가 들어오는 race condition을 막기 위해 DB unique constraint와 원자적 처리가 필요합니다.

### Q3. At-least-once 전달에서는 무엇이 필요한가요?

> 메시지가 중복 전달될 수 있다는 전제로 consumer를 idempotent하게 만들어야 합니다. message id를 저장하고 unique constraint를 사용해 중복 처리를 막는 방식이 대표적입니다.

### Q4. Exactly-once가 왜 어려운가요?

> 네트워크 장애 때문에 실제 비즈니스 처리는 완료됐지만 ACK나 응답이 전달되지 않을 수 있기 때문입니다. 송신자는 처리 여부를 확실히 알 수 없어서 재전송할 수 있고, 이 때문에 중복이 발생합니다. 그래서 실무에서는 at-least-once 전달과 idempotent consumer로 exactly-once에 가까운 효과를 만드는 경우가 많습니다.

### Q5. Retry할 수 없는 연산도 있나요?

> 비-idempotent한 side effect를 가진 연산은 무작정 retry하면 위험합니다. 결제, 주문 생성, 외부 메시지 전송 등은 idempotency key, deduplication, transaction 또는 외부 시스템의 중복 방지 기능을 같이 설계해야 합니다.

---

## 17. 60초 답변

> 분산 시스템에서는 요청을 처리했지만 응답이나 ACK가 유실될 수 있기 때문에 재시도가 중복 side effect를 만들 수 있습니다. 그래서 idempotency와 deduplication이 중요합니다. API에서는 idempotency key를 받아 처리 결과를 저장하고 같은 키가 다시 오면 기존 결과를 반환할 수 있습니다. 메시지 큐에서는 message id를 저장하고 unique constraint로 중복 처리를 막을 수 있습니다. 완전한 exactly-once 보장은 어렵기 때문에 실무에서는 at-least-once 전달을 허용하고 consumer를 idempotent하게 설계해서 exactly-once에 가까운 효과를 만드는 방식이 흔합니다.

---

## 18. 기억할 핵심

```text
1. 네트워크 실패는 "실패"가 아니라 "결과를 모름"일 수 있다.
2. Retry는 중복 side effect를 만들 수 있다.
3. Idempotency key는 POST 재시도를 안전하게 만들 수 있다.
4. Unique constraint는 race condition 방지에 중요하다.
5. At-least-once에서는 중복 전달을 전제로 해야 한다.
6. Exactly-once는 전체 분산 시스템에서 단순하게 보장하기 어렵다.
7. 실무에서는 idempotency + deduplication으로 exactly-once effect를 만든다.
8. Transactional Outbox와 Idempotent Consumer는 함께 자주 사용된다.
```

### 최종 한 문장

> **분산 시스템에서는 중복이 안 온다고 믿기보다, 중복이 와도 안전하도록 설계하는 것이 더 현실적이다.**
