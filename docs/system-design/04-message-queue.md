# 04. Message Queue

Message Queue의 핵심은 **요청을 즉시 끝내야 하는 일과 나중에 처리해도 되는 일을 분리하고, 생산자와 소비자의 속도 차이를 흡수하는 것**이다.

## 1. 비유

식당 주문표를 생각해보자.

손님이 주문할 때마다 요리사가 직접 주문을 받으면 주문 접수와 요리가 강하게 묶인다.

주문표를 중간에 쌓아두면:
- 주문은 빨리 접수하고
- 주방은 가능한 속도로 처리할 수 있다.

Queue가 이 완충 역할을 한다.

## 2. Synchronous vs Asynchronous Processing

### Synchronous
```text
Client → API → Email Service → Payment → Response
```

모든 downstream이 끝날 때까지 기다린다.

### Asynchronous
```text
Client → API → Queue → Response
                  ↓
                Worker
```

요청 핵심 작업만 끝내고 나머지는 worker가 처리한다.

## 3. Queue를 쓰는 이유

- latency 분리
- spike 흡수
- producer/consumer decoupling
- retry
- background processing
- fan-out
- batch 처리

## 4. 예시: 이미지 변환

대용량 TIFF→PDF 변환 요청이 20초 걸린다면 HTTP request thread를 계속 붙잡는 대신:

```text
POST /conversion
→ Job row 생성
→ Queue publish
→ 202 Accepted

Worker
→ TIFF→PDF
→ Object Storage 저장
→ Job status 완료
```

이 구조는 긴 작업에 특히 자연스럽다.

## 5. At-most-once / At-least-once / Exactly-once

### At-most-once
중복은 줄지만 메시지를 잃을 수 있다.

### At-least-once
메시지를 잃지 않기 위해 재전달할 수 있다.
따라서 **중복 처리 가능성**이 있다.

### Exactly-once
끝단 비즈니스 side effect까지 완벽히 exactly once를 보장하는 것은 어렵다.

실무에서는 보통:
> at-least-once delivery + idempotent consumer

조합이 중요하다.

## 6. Acknowledgement

Consumer가 메시지를 성공적으로 처리했음을 broker에 알려준다.

Ack 전에 consumer가 죽으면 재전달될 수 있다.

그래서 다음 같은 순서가 중요하다.

```text
message receive
→ business transaction
→ commit
→ ack
```

하지만 broker와 DB는 하나의 local transaction이 아니므로 partial failure를 고려해야 한다.

## 7. Retry와 Dead Letter Queue

실패 메시지를 무한 retry하면 시스템을 더 망가뜨릴 수 있다.

보통:
- retry count 제한
- exponential backoff
- jitter
- DLQ(Dead Letter Queue)

를 둔다.

DLQ는 실패를 숨기는 곳이 아니라 **운영자가 원인을 분석하고 재처리하는 격리 공간**이다.

## 8. Ordering

Queue가 전체 메시지의 완벽한 순서를 보장한다고 가정하면 안 된다.

분산 처리에서는 보통:
- partition/key 단위 ordering
- 동일 aggregate를 같은 partition으로 routing

같은 전략을 쓴다.

예:
```text
accountId → same partition
```

## 9. Consumer Scaling

Queue backlog가 늘어나면 consumer 수를 늘릴 수 있다.

하지만 무작정 늘리면 downstream DB가 먼저 죽을 수 있다.

즉:
```text
Queue Consumer 100개
→ DB Connection 100개 이상
→ DB saturation
```

따라서 consumer concurrency는 downstream capacity 기준으로 정해야 한다.

## 10. Backpressure와 Queue Depth

Queue depth가 계속 증가한다는 것은 producer가 consumer보다 빠르다는 뜻이다.

중요 지표:
- queue length
- oldest message age
- consume rate
- publish rate
- retry rate
- DLQ size

Queue 자체가 해결책이 아니라 **지연을 저장하는 장소**가 될 수도 있다.

## 11. Queue vs Pub/Sub

### Queue
보통 하나의 작업을 하나의 consumer가 처리하는 work distribution에 적합하다.

### Pub/Sub
하나의 event를 여러 subscriber가 각각 받는 fan-out에 적합하다.

예:
```text
OrderCreated
├─ Email
├─ Analytics
└─ Inventory
```

제품별로 semantics는 다르므로 이름만 보고 단정하면 안 된다.

## 12. Message Broker가 Source of Truth인가

일반적으로 비즈니스 상태의 최종 source of truth는 DB인 경우가 많다.

DB write와 message publish를 따로 하면 dual-write 문제가 생긴다.

그래서 Database에서 배운:
- Transactional Outbox
- CDC

가 연결된다.

## 13. Poison Message

항상 실패하는 특정 메시지를 poison message라고 부를 수 있다.

무한 재시도하면 consumer 전체 처리량을 갉아먹는다.

따라서:
- retry 제한
- DLQ
- validation
- observability

가 필요하다.

## 14. Queue의 비용

Queue를 넣으면:
- eventual consistency
- 운영 복잡도
- duplicate handling
- ordering 문제
- observability 필요

가 추가된다.

작은 CRUD 시스템에 무조건 Kafka가 필요한 것은 아니다.

## 15. 60초 면접 답변

> Message Queue는 producer와 consumer를 시간적으로 분리해서 요청 latency를 줄이고 traffic spike를 흡수하는 완충 계층입니다. 긴 background job, email, image conversion 같은 작업에 적합합니다. 다만 queue를 쓰면 eventual consistency, 중복 처리, ordering, retry 같은 문제가 생깁니다. 실무에서는 at-least-once delivery를 전제로 consumer를 idempotent하게 만들고, retry에는 backoff와 DLQ를 둡니다. 또 queue backlog가 늘어난다고 consumer를 무작정 늘리면 downstream DB를 과부하시킬 수 있으므로 queue depth와 oldest message age, consume rate를 보면서 downstream capacity에 맞춰 concurrency를 조절해야 합니다.
