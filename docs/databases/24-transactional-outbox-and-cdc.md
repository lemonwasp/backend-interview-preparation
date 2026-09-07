# 24. Transactional Outbox / CDC

## 한 줄 요약

Transactional Outbox는 **DB 변경과 Event 발행 의도를 같은 Local Transaction에 기록**해서 DB와 Message Broker 사이의 dual-write 문제를 줄이는 패턴이고, CDC(Change Data Capture)는 DB의 변경 Log를 읽어 다른 시스템으로 전달하는 방식입니다.

---

## 왜 Dual Write가 문제일까?

예를 들어 주문 생성 API가:

1. DB에 Order INSERT
2. Kafka에 `OrderCreated` publish

를 수행한다고 합시다.

문제는 두 작업이 서로 다른 시스템이라는 점입니다.

### 실패 A

```text
DB COMMIT 성공
→ Kafka publish 실패
```

결과:

- 주문은 존재
- Event는 없음

### 실패 B

```text
Kafka publish 성공
→ DB COMMIT 실패
```

결과:

- Event는 존재
- 실제 주문은 없음

Local DB Transaction 하나로 두 시스템을 완전히 Atomic하게 묶을 수는 없습니다.

---

# Transactional Outbox

핵심 아이디어는:

> Business Row와 "발행해야 할 Event"를 같은 DB Transaction에 저장한다.

예:

```sql
BEGIN;

INSERT INTO orders (...);

INSERT INTO outbox (
    event_id,
    event_type,
    aggregate_id,
    payload,
    created_at
) VALUES (...);

COMMIT;
```

이제 Order와 Outbox Row는 같이 Commit되거나 같이 Rollback됩니다.

---

## 그 다음 Event는 누가 보내나?

별도 Relay/Worker가 Outbox Table을 읽고 Kafka 같은 Broker로 전송합니다.

```text
Application Transaction
  ↓
orders + outbox commit
  ↓
Relay
  ↓
Message Broker
```

---

## 그래도 중복 발행은 가능하다

Relay가:

1. Broker publish 성공
2. `published=true` 저장 전에 Crash

하면 재시작 후 같은 Event를 다시 publish할 수 있습니다.

따라서:

> Outbox는 보통 at-least-once delivery와 함께 생각해야 한다.

Consumer는 Event ID 기반 Deduplication 또는 Idempotent 처리를 고려해야 합니다.

---

## Exactly-once라는 말에 주의

End-to-end 시스템 전체에서 exactly-once Side Effect를 보장하는 것은 매우 어렵습니다.

Broker의 exactly-once 기능이 있어도:

- External API
- Email
- Payment
- 다른 DB

까지 자동으로 exactly-once가 되는 것은 아닙니다.

면접에서는:

> 기술 하나의 exactly-once와 비즈니스 Side Effect 전체의 exactly-once를 구분해야 한다.

---

# CDC — Change Data Capture

CDC는 DB의 변경 내용을 감지해 외부 시스템으로 전달하는 방식입니다.

대표적인 접근:

- WAL / Binlog / Transaction Log 읽기
- Trigger 기반
- Timestamp polling

실무에서는 Log-based CDC가 중요한 방식입니다.

예:

```text
Database WAL/Binlog
  ↓
CDC Connector
  ↓
Kafka
  ↓
Search / Analytics / Cache / other services
```

---

## Outbox + CDC

Outbox Table을 Application이 Transaction 안에서 기록하고,
CDC Connector가 Outbox 변경을 읽어 Broker로 전달하는 구조를 사용할 수 있습니다.

장점:

- Polling Relay 부담 감소 가능
- DB Transaction과 Event intent 결합

하지만 운영 복잡도는 여전히 존재합니다.

---

# Ordering 문제

Event A와 B 순서가 중요할 수 있습니다.

예:

```text
OrderCreated
OrderCancelled
```

Partitioning key, aggregate ID, sequence/version 등을 잘못 설계하면 순서가 뒤섞일 수 있습니다.

따라서:

- 같은 Aggregate의 Event routing
- version / sequence number
- stale event 처리

를 고려합니다.

---

# Consumer Idempotency

Consumer가 Event를 중복 수신할 수 있으므로:

```sql
INSERT INTO processed_events(event_id)
VALUES (?);
```

같은 Unique Constraint를 이용하거나, Business Operation 자체를 idempotent하게 설계할 수 있습니다.

---

# Outbox Table 운영 이슈

Outbox가 계속 쌓이면:

- Table 증가
- Index 증가
- Scan 비용
- Cleanup 비용

이 생깁니다.

따라서:

- 상태/시간 Index
- Batch 처리
- Archive / Delete
- retention

을 설계해야 합니다.

---

# Outbox가 만능은 아니다

Outbox는:

- DB + Broker dual write

문제를 줄이는 데 유용하지만:

- Consumer business logic 실패
- Cross-service transaction
- Event ordering
- duplicate delivery
- poison message

를 자동으로 해결하지는 않습니다.

---

## 60초 면접 답변

> DB에 데이터를 저장하고 Kafka에 Event를 발행하는 두 작업을 단순히 순서대로 실행하면 한쪽만 성공하는 dual-write 문제가 생깁니다. Transactional Outbox는 Business Data와 Outbox Event Row를 같은 Local DB Transaction 안에서 저장해서 "데이터는 저장됐는데 Event 발행 의도가 사라지는" 문제를 줄입니다. 이후 Relay나 CDC가 Outbox를 Broker로 전달합니다. 다만 Relay Crash 등으로 Event가 중복 발행될 수 있으므로 보통 at-least-once delivery를 전제로 Consumer Idempotency가 필요합니다. CDC는 WAL이나 Binlog 같은 DB 변경 Log를 읽어 변경사항을 외부 시스템으로 전달하는 메커니즘입니다.

---

## 백엔드 연결 포인트

- Event-driven Architecture
- Kafka
- Dual Write
- Idempotency
- At-least-once
- CDC
- Search Index sync
- Cache invalidation

다음: **25. Database Review / Mock Interview**
