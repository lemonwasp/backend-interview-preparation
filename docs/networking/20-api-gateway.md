# 20. API Gateway

## 학습 목표

- API Gateway가 무엇인지 설명한다.
- Reverse Proxy, Load Balancer와 차이를 설명한다.
- 인증, Routing, Rate Limiting, observability 같은 공통 기능을 어디까지 맡길지 이해한다.
- Gateway가 Single Point of Failure나 Bottleneck이 될 수 있음을 설명한다.

---

## 1. 가장 쉬운 설명

API Gateway는 여러 Backend Service 앞에 놓이는 공통 진입점입니다.

```text
Client
  │
  ▼
API Gateway
  ├── User Service
  ├── Order Service
  └── Payment Service
```

Client는 내부 서비스 주소를 직접 알 필요 없이 Gateway 하나를 바라볼 수 있습니다.

즉,

> API Gateway = 외부 요청을 내부 서비스로 연결하면서 공통 정책을 적용하는 진입 계층

입니다.

---

## 2. 주요 역할

대표 기능:

- Routing
- Authentication / Authorization 보조
- TLS termination
- Rate Limiting
- Request/Response transformation
- API version routing
- Logging / Metrics / Tracing
- CORS 처리
- Request size limit
- WAF 연계
- Canary / traffic split

하지만 모든 비즈니스 로직을 Gateway에 넣으면 안 됩니다.

---

## 3. Reverse Proxy와 차이

API Gateway는 본질적으로 Reverse Proxy 역할을 포함할 수 있습니다.

하지만 보통 API Gateway는 API-specific 정책이 더 강합니다.

| 항목 | Reverse Proxy | API Gateway |
|---|---|---|
| Reverse routing | O | O |
| TLS termination | O | O |
| Static serving/cache | 자주 사용 | 가능 |
| Authentication policy | 가능 | 핵심 기능인 경우 많음 |
| Rate limiting | 가능 | 흔함 |
| API key / quota | 덜 중심 | 흔함 |
| Service aggregation | 제한적 | 가능 |

경계는 제품마다 겹칩니다.

---

## 4. Load Balancer와 차이

Load Balancer의 핵심은 여러 backend instance로 traffic을 분산하는 것입니다.

API Gateway의 핵심은 API 진입 정책과 service routing입니다.

실제 구조에서는 둘이 함께 존재할 수 있습니다.

```text
Internet
  │
  ▼
L4/L7 Load Balancer
  │
  ▼
API Gateway cluster
  │
  ├── Service A
  └── Service B
```

또는 cloud product 하나가 여러 기능을 동시에 제공할 수도 있습니다.

---

## 5. Authentication을 Gateway에서 끝내도 되는가?

Gateway에서 token signature를 검증하면 각 서비스의 중복 작업을 줄일 수 있습니다.

하지만 이것이 내부 서비스가 authorization을 전혀 하지 않아도 된다는 뜻은 아닙니다.

예:

- Gateway: 이 사용자가 유효한 로그인 사용자인가?
- Order Service: 이 사용자가 이 주문을 조회할 권한이 있는가?

즉 Authentication과 resource-level Authorization 책임을 구분해야 합니다.

---

## 6. Rate Limiting

Gateway는 외부 요청의 공통 진입점이므로 Rate Limit을 적용하기 좋은 위치입니다.

예:

```text
100 requests / minute / API key
```

목적:

- abuse 방지
- backend 보호
- quota 구현
- 비용 통제

하지만 여러 Gateway instance가 있을 때는 분산된 counter를 어떻게 일관되게 관리할지 고려해야 합니다.

---

## 7. Request Aggregation

Mobile Client가 화면 하나를 위해 5개 서비스를 호출한다고 합시다.

Gateway 또는 BFF가 내부 호출을 합쳐 한 응답을 만들 수도 있습니다.

```text
Client → Gateway
           ├→ User
           ├→ Order
           └→ Recommendation
```

장점:

- Client round trip 감소

단점:

- Gateway에 business orchestration이 과도하게 쌓일 수 있음
- downstream 하나의 지연이 전체 응답을 느리게 할 수 있음

복잡한 orchestration은 별도 BFF/Service가 더 적합할 수 있습니다.

---

## 8. Gateway가 Bottleneck이 되는 이유

모든 요청이 Gateway를 통과하면:

- CPU
- TLS
- connection
- logging
- authentication
- transformation

비용이 집중됩니다.

따라서 Gateway 자체도 horizontally scalable해야 하고 high availability가 필요합니다.

---

## 9. Single Point of Failure 방지

```text
           ┌─ Gateway 1
Load Balancer
           └─ Gateway 2
```

여러 instance와 health check를 둡니다.

또한 config 변경 실패, certificate 문제, rate-limit storage 장애처럼 Gateway 계층 자체의 운영 리스크도 고려해야 합니다.

---

## 10. Timeout / Retry를 어디서 할까?

Gateway에서 downstream timeout을 설정하는 것은 중요합니다.

하지만 무분별한 Retry는 위험합니다.

예:

```text
Client retry × Gateway retry × Service retry
```

Retry amplification이 발생할 수 있습니다.

따라서 전체 request budget과 idempotency를 고려해 retry 위치와 횟수를 설계해야 합니다.

이 내용은 다음 Retry/Timeout/Circuit Breaker 파트와 연결됩니다.

---

## 11. Observability

Gateway는 모든 외부 요청이 지나가기 때문에 다음 데이터를 수집하기 좋습니다.

- request count
- status code
- latency
- route
- client/application ID
- trace ID

하지만 Gateway 로그만으로 내부 병목 원인을 알 수 없으므로 distributed tracing이 중요합니다.

---

## 12. API Gateway vs Service Mesh

API Gateway는 주로 **north-south traffic**, 즉 외부 Client와 내부 시스템 경계에 집중합니다.

Service Mesh는 주로 **east-west traffic**, 즉 내부 service-to-service 통신에 집중합니다.

완전히 절대적인 구분은 아니지만 면접에서 유용한 기본 프레임입니다.

---

## 13. 자주 하는 오해

### Gateway에 모든 공통 로직을 넣으면 좋다

과도하면 Gateway가 거대한 monolith가 됩니다.

### Gateway가 authorization을 하면 backend는 보안을 신경 쓸 필요가 없다

resource-level authorization은 domain service가 책임져야 할 수 있습니다.

### Gateway 하나만 두면 된다

그 자체가 SPOF가 될 수 있습니다.

---

## 14. 60초 면접 답변

> API Gateway는 외부 Client 요청의 공통 진입점으로서 내부 서비스 routing과 authentication, rate limiting, TLS termination, logging 같은 cross-cutting concern을 처리하는 계층입니다. Reverse Proxy와 기능이 겹치지만 API-specific 정책과 quota, version routing 같은 기능이 더 강조됩니다. 모든 요청이 지나기 때문에 bottleneck이나 single point of failure가 될 수 있어 Gateway 자체도 여러 instance로 확장하고 health check와 observability를 갖춰야 합니다. 또한 모든 business logic을 Gateway에 넣기보다 domain authorization이나 orchestration은 backend/BFF와 적절히 분리해야 합니다.
