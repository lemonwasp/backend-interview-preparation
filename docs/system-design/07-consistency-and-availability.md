# 07. Consistency / Availability

## 한 줄 정의

분산 시스템에서 Consistency와 Availability는 **장애와 네트워크 지연이 있는 상황에서 어떤 보장을 우선할지 결정하는 설계 선택**이다.

## 먼저 중요한 점

`Consistency가 좋다`와 `Availability가 좋다`는 추상적인 칭찬이 아니다.

각 데이터와 사용자 흐름마다 필요한 보장이 다르다.

예:

- 은행 잔액: 잘못된 최신 상태를 보여주면 치명적일 수 있다.
- 좋아요 수: 수 초 늦게 반영돼도 괜찮을 수 있다.

즉 시스템 전체를 한 단어로 `강한 일관성` 또는 `최종 일관성`이라고 부르기보다, **어떤 데이터와 operation에 어떤 보장이 필요한지**를 정해야 한다.

---

## 1. Consistency란 무엇인가

이 문맥에서 Consistency는 여러 복제본이나 노드가 있을 때 사용자가 보는 데이터의 일관된 상태와 관련된다.

대표적인 보장:

- Strong Consistency
- Eventual Consistency
- Read-after-write Consistency
- Monotonic Read

### Strong Consistency

성공한 쓰기 이후의 읽기는 최신 상태를 보는 식의 강한 보장을 목표로 한다.

대가:

- coordination
- quorum/leader 통신
- latency 증가 가능
- 일부 장애에서 요청 거절 가능

### Eventual Consistency

즉시 같지 않아도 시간이 지나면 복제본이 수렴하는 모델이다.

장점:

- 높은 가용성/확장성에 유리한 경우가 많다.

대가:

- stale read
- conflict
- 보상 UX/재처리 필요

---

## 2. Availability란 무엇인가

여기서는 요청을 받을 수 있고 응답을 제공할 수 있는 능력을 뜻한다.

중요한 점:

- 단순 `서버가 켜져 있다`와는 다르다.
- 서비스 전체 availability는 dependency chain의 영향을 받는다.

예:

```text
API -> Payment -> DB -> External Fraud API
```

하나의 필수 dependency가 실패하면 사용자 관점 availability도 떨어질 수 있다.

---

## 3. CAP Theorem을 정확히 이해하기

CAP은 다음 세 단어를 말한다.

- C: Consistency
- A: Availability
- P: Partition Tolerance

핵심은 **네트워크 partition이 실제로 발생했을 때 C와 A를 동시에 완벽히 보장할 수 없다는 것**이다.

따라서 `CAP에서 3개 중 평소 2개를 고른다`는 식으로만 외우면 부정확하다.

분산 시스템에서는 network partition을 무시할 수 없으므로, partition 상황에서 어떤 요청을 거절하고 어떤 요청을 계속 받을지 결정해야 한다.

---

## 4. CP와 AP를 실무적으로 보기

### Consistency 우선

예:

- 재고 마지막 1개 예약
- 금융 이체
- leader election metadata

partition 상황에서 최신 상태 확인이 안 되면 쓰기를 거절하는 쪽을 선택할 수 있다.

### Availability 우선

예:

- 좋아요 이벤트 수집
- 일부 feed 조회
- analytics event

일시적으로 서로 다른 상태를 허용하고 나중에 reconcile할 수 있다.

단, 실제 제품은 `서비스 전체가 CP/AP`라기보다 operation별 선택이 섞여 있는 경우가 많다.

---

## 5. Quorum 개념

Replication factor가 N이고:

- Write quorum = W
- Read quorum = R

일부 시스템에서는 `R + W > N` 같은 조건을 이용해 read/write set이 겹치도록 설계할 수 있다.

하지만 이것만으로 모든 실제 consistency 문제가 자동 해결되는 것은 아니다.

- versioning
- conflict resolution
- leader/replica behavior
- failure detection

등 구현 세부사항이 중요하다.

---

## 6. Read-after-write Consistency

사용자가 직접 수정한 직후에는 최신 값이 보이길 기대하는 경우가 많다.

예:

```text
POST /profile
GET /profile
```

쓰기 후 GET을 replica로 보내면 lag 때문에 이전 값이 보일 수 있다.

대안:

- 일정 시간 Primary read
- session/user sticky read
- version/token 기반 routing
- replica가 특정 log position을 따라잡을 때까지 대기

제품 요구에 따라 비용과 UX를 선택한다.

---

## 7. Eventual Consistency에는 보상 설계가 필요하다

`나중에 맞아진다`는 말만으로는 부족하다.

확인할 것:

- 최대 지연을 어느 정도 허용하는가?
- 중복 이벤트는 어떻게 처리하는가?
- 순서가 바뀌면 어떻게 하는가?
- 실패한 동기화는 어떻게 재시도하는가?
- 사용자가 중간 상태를 보아도 되는가?

예:

주문 상태:

```text
PAYMENT_CONFIRMED
      ↓
ORDER_PAID
      ↓
INVENTORY_RESERVED
```

각 단계가 비동기라면 중간 상태와 실패 복구를 제품 차원에서 설계해야 한다.

---

## 8. Consistency와 Database Isolation은 다른 축이다

혼동하기 쉽다.

### Transaction Isolation

한 DB 안에서 concurrent transaction이 서로 어떻게 보이는지 다룬다.

### Distributed Consistency

복수 노드/복제본/서비스 사이에서 상태가 어떻게 일치하는지 다룬다.

둘 다 `일관성`이라는 단어가 나오지만 같은 문제가 아니다.

---

## 9. 흔한 오해

### 오해 1: Strong Consistency가 항상 더 좋은 설계다

아니다. latency와 availability 비용이 있고 모든 데이터에 필요하지 않다.

### 오해 2: Eventual Consistency는 데이터가 틀려도 된다는 뜻이다

아니다. 일시적인 불일치를 허용하되 결국 정의된 상태로 수렴해야 한다.

### 오해 3: CAP은 무조건 세 가지 중 두 가지를 고르는 문제다

partition이 발생한 상황에서 C와 A의 trade-off가 핵심이다.

### 오해 4: Replica가 있으면 항상 읽기 가능하다

최신성 요구가 강하면 stale replica의 응답을 허용할 수 없을 수 있다.

---

## Backend 설계 연결

좋은 답변은 다음 식으로 요구사항을 나눈다.

| 데이터 | 요구 |
|---|---|
| 결제 상태 | 강한 정합성 / 중복 방지 중요 |
| 상품 상세 | 수 초 stale 허용 가능 |
| 좋아요 수 | eventual 가능 |
| 내 프로필 수정 직후 | read-after-write 선호 |
| 분석 이벤트 | availability 우선 가능 |

---

## 60초 면접 답변

> Consistency와 Availability는 분산 시스템에서 장애와 네트워크 partition이 있을 때 어떤 보장을 우선할지 정하는 문제입니다. CAP의 핵심은 partition 상황에서 강한 consistency와 모든 요청에 대한 availability를 동시에 완벽히 보장할 수 없다는 점입니다. 그래서 데이터마다 요구사항을 나눠야 합니다. 예를 들어 결제나 마지막 재고 예약은 최신 상태 확인이 중요해서 consistency를 우선할 수 있고, 좋아요 수나 analytics는 일시적인 stale state를 허용하고 eventual consistency를 선택할 수 있습니다. 또한 사용자가 직접 수정한 직후에는 read-after-write consistency를 별도로 제공하는 식으로 operation별 보장을 설계합니다.