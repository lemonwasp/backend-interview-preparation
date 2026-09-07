# 26. Idempotency Key

## 한 줄 요약

Idempotency Key는 같은 요청이 여러 번 도착해도 서버가 **한 번만 처리한 것처럼** 만들기 위한 요청 식별자다.

## 왜 필요한가

네트워크에서는 응답을 못 받았다고 해서 서버 처리가 실패했다고 단정할 수 없다.

예를 들어 결제 요청을 보낸 뒤 Client가 Timeout을 받았다.

```text
Client -> Payment API: 결제 10,000원
Server: 결제 성공
Server -> Client: 응답 전송 중 연결 끊김
Client: 실패했다고 생각하고 Retry
```

두 번째 요청까지 그대로 처리하면 결제가 두 번 일어날 수 있다.

핵심 문제는 이것이다.

> 요청 실패와 응답 수신 실패는 같은 사건이 아니다.

## Idempotent란

같은 연산을 여러 번 수행해도 최종 결과가 한 번 수행했을 때와 같으면 Idempotent하다고 한다.

예:

- `PUT /users/1/name = Hong`은 보통 반복해도 결과가 같다.
- `DELETE /users/1`도 상태 관점에서는 반복해도 삭제 상태가 유지될 수 있다.
- `POST /payments`는 매번 새 결제를 만들 수 있으므로 기본적으로 안전한 Retry 대상이 아니다.

HTTP Method만 보고 완전히 판단하면 안 된다. 실제 비즈니스 의미가 더 중요하다.

## Idempotency Key 흐름

Client가 요청마다 고유 Key를 만든다.

```http
POST /payments
Idempotency-Key: 2f3a7b...
```

Server는 보통 다음처럼 처리한다.

```text
1. Key 조회
2. 처음 보는 Key라면 처리 시작
3. 결과를 Key와 함께 저장
4. 같은 Key가 다시 오면 새 처리를 하지 않고 기존 결과 반환
```

## 단순한 구현 모델

```text
IdempotencyKey
    -> status: PROCESSING | SUCCEEDED | FAILED
    -> request fingerprint
    -> response/result reference
    -> created_at / expires_at
```

### 왜 PROCESSING 상태가 필요한가

동시에 같은 Key가 두 요청으로 들어올 수 있다.

```text
Request A ----\
              -> same key
Request B ----/
```

`SELECT 후 INSERT`만 단순하게 하면 Race Condition이 생길 수 있다.

따라서 DB Unique Constraint, Atomic Insert, Lock 등으로 "이 Key의 최초 처리자"를 하나로 정해야 한다.

## Request Fingerprint

같은 Key인데 요청 내용이 다르면 위험하다.

```text
Key = abc
첫 요청: 10,000원 결제
두 번째: 100,000원 결제
```

그래서 서버는 Key뿐 아니라 핵심 요청 Payload의 Hash/Fingerprint를 함께 저장할 수 있다.

같은 Key + 다른 Payload라면 보통 오류로 거절한다.

## 결과를 얼마나 저장할까

대표적인 선택지는 두 가지다.

1. 전체 Response 저장
2. 생성된 Resource ID 등 최소 결과만 저장하고 다시 조회

전체 Response 저장은 재응답이 쉽지만 저장 비용이 크다.
최소 결과 저장은 공간은 적게 쓰지만 재구성 로직이 필요하다.

## TTL

Idempotency Key를 영원히 저장할 수는 없다.

그래서 TTL을 둔다.

TTL은 다음과 관련 있다.

- Client가 실제로 Retry할 수 있는 최대 시간
- 비즈니스 중복 처리 위험 기간
- 저장 비용

TTL이 너무 짧으면 늦은 Retry가 새 요청처럼 처리될 수 있다.

## Retry와 함께 생각하기

Idempotency Key는 Retry 자체를 안전하게 만들어주는 도구다.

```text
Timeout
  -> Retry 필요 여부 판단
  -> Non-idempotent operation인가?
  -> Idempotency Key로 중복 처리 방지
  -> Backoff + Jitter 적용
```

## Exactly-once인가

엄밀히 말해 분산 시스템에서 "완벽한 exactly-once"는 단순한 Key 하나로 해결되지 않는다.

Idempotency Key는 주로 **중복 요청의 효과를 억제하여 effectively-once에 가까운 비즈니스 동작**을 만드는 패턴으로 이해하는 것이 안전하다.

DB Commit, 외부 API 호출, Message Publish가 한 요청 안에 섞이면 추가적인 Transaction/Outbox/Saga 설계가 필요할 수 있다.

## Backend 면접 연결

### 결제 API

`POST /payments`에 Idempotency Key를 받아 중복 결제를 막는다.

### 주문 생성

모바일 네트워크가 불안정한 상황에서 사용자가 주문 버튼을 여러 번 눌러도 주문 하나만 만들어지도록 한다.

### 외부 API Retry

외부 시스템이 Idempotency Key를 지원하지 않는다면 Client 측 Retry를 매우 조심해야 한다.

## 흔한 오해

### "UUID만 붙이면 끝이다"

아니다. Server가 Key를 Atomic하게 기록하고 결과를 재사용해야 한다.

### "같은 Key면 무조건 같은 응답"

요청 Payload가 달라지면 충돌로 판단해야 할 수 있다.

### "Idempotency Key면 Exactly-once 보장"

아니다. 범위와 저장 경계를 명확히 해야 한다.

## 60초 면접 답변

> Idempotency Key는 재시도 가능한 API에서 같은 비즈니스 작업이 중복 실행되는 것을 막기 위한 고유 요청 식별자입니다. 서버는 최초 요청에서 Key와 처리 결과를 저장하고 같은 Key가 다시 오면 새 작업을 실행하지 않고 기존 결과를 반환합니다. 결제나 주문 생성처럼 POST Retry가 중복 부작용을 만들 수 있는 경우 특히 중요합니다. 구현할 때는 Key의 Unique Constraint나 Atomic Insert로 동시 요청을 막고, 같은 Key에 다른 Payload가 들어오는 문제를 막기 위해 Request Fingerprint를 함께 저장할 수 있습니다. 또한 TTL과 저장 범위를 정해야 하며 Idempotency Key 자체가 분산 시스템 전체의 Exactly-once를 자동으로 보장하는 것은 아닙니다.
