# 19. REST vs RPC

## 학습 목표

- REST와 RPC가 무엇을 중심으로 설계되는지 설명한다.
- Resource-oriented와 Action-oriented 차이를 이해한다.
- public API와 service-to-service API에서 선택 기준을 설명한다.
- REST와 RPC를 단순히 JSON vs Protobuf로 오해하지 않는다.

---

## 1. 가장 쉬운 설명

REST는 주로 **Resource**를 중심으로 생각합니다.

```text
GET /users/123
POST /orders
DELETE /sessions/abc
```

RPC는 주로 **Action/Method**를 중심으로 생각합니다.

```text
GetUser(123)
CreateOrder(...)
CancelPayment(...)
```

핵심 차이는 wire format보다 API를 바라보는 추상화입니다.

---

## 2. REST는 HTTP semantics를 적극 사용한다

RESTful API에서는 HTTP method와 status code를 의미 있게 사용합니다.

- GET: 조회
- POST: 생성/처리
- PUT: 전체 대체
- PATCH: 부분 수정
- DELETE: 삭제

그리고 200, 201, 204, 400, 404, 409 같은 status code를 활용합니다.

이 덕분에 browser, proxy, cache, CDN 같은 HTTP 생태계와 잘 맞습니다.

---

## 3. RPC는 business operation을 직접 드러낸다

RPC는 복잡한 command를 표현하기 쉽습니다.

예:

```text
ApproveLoan()
RecalculatePortfolio()
GenerateMonthlyStatement()
```

REST에서도 표현할 수 있지만 억지로 CRUD resource로만 모델링하면 오히려 부자연스러워질 수 있습니다.

---

## 4. REST = JSON, RPC = Binary가 아니다

이건 흔한 오해입니다.

REST는 JSON을 많이 쓰지만 XML이나 다른 representation도 가능합니다.

RPC도 JSON-RPC처럼 JSON을 사용할 수 있습니다.

gRPC가 Protobuf를 많이 사용하는 것은 구현 선택이지 REST/RPC의 본질적 정의가 아닙니다.

---

## 5. Coupling

RPC는 client가 server의 method contract를 직접 아는 느낌이 강합니다.

REST는 resource와 standard HTTP semantics를 통해 상대적으로 느슨한 coupling을 만들 수 있습니다.

하지만 실제 coupling은 API versioning, schema, behavior contract에 크게 좌우되므로 REST라고 자동으로 loose coupling이 되는 것은 아닙니다.

---

## 6. Idempotency

REST에서는 HTTP method semantics를 활용할 수 있습니다.

예:

- GET: idempotent
- PUT: 일반적으로 idempotent
- DELETE: 의미적으로 idempotent
- POST: 일반적으로 non-idempotent

RPC에서는 method마다 idempotency를 별도로 정의해야 합니다.

Retry 설계에서 이 차이가 중요합니다.

---

## 7. Public API에서 REST가 자주 쓰이는 이유

- HTTP tooling이 풍부함
- curl/browser로 테스트 쉬움
- JSON이 사람이 읽기 쉬움
- CDN/cache/proxy와 잘 맞음
- 다양한 client가 접근하기 쉬움

---

## 8. Internal RPC에서 gRPC가 자주 쓰이는 이유

- 명확한 schema
- code generation
- strongly typed contract
- compact payload
- streaming
- 내부 서비스 간 높은 호출 빈도

---

## 9. 선택 기준

### REST가 잘 맞는 경우

- 외부 공개 API
- CRUD/resource 중심
- browser/client compatibility 중요
- HTTP cache 활용 중요

### RPC가 잘 맞는 경우

- 내부 service-to-service
- operation 중심
- strict schema 중요
- streaming 필요
- code generation 활용

---

## 10. Backend 예시

외부:

```text
Mobile App → REST/JSON → API Gateway
```

내부:

```text
API Gateway → Order Service → gRPC → Payment Service
```

한 시스템에서 REST와 RPC를 함께 쓰는 것은 자연스럽습니다.

---

## 11. 자주 하는 오해

### REST가 항상 더 느리다

아닙니다. 전체 latency는 DB와 downstream이 더 크게 좌우할 수 있습니다.

### RPC가 항상 더 결합도가 높다

대체로 method contract coupling이 강하지만 versioning discipline에 따라 달라집니다.

### REST는 CRUD만 해야 한다

아닙니다. Resource modeling과 HTTP semantics를 중심으로 설계할 뿐입니다.

---

## 12. 60초 면접 답변

> REST는 resource와 HTTP semantics를 중심으로 API를 설계하고, RPC는 remote operation이나 method 호출을 중심으로 설계합니다. REST는 public API에서 HTTP tooling, cache, proxy, browser compatibility가 좋아 자주 사용되고, gRPC 같은 RPC는 내부 서비스 통신에서 strict schema, code generation, streaming과 효율적인 payload가 장점입니다. REST와 RPC의 차이를 JSON 대 Protobuf로 보면 안 되고, 핵심은 API 추상화와 contract 모델입니다. 실제 시스템에서는 외부에는 REST, 내부에는 gRPC를 함께 사용하는 구조도 흔합니다.
