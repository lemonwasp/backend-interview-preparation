# 02. Stateless Service

Stateless Service의 핵심은 **어떤 서버 인스턴스로 요청이 가더라도 처리할 수 있게 만드는 것**이다.

## 1. 비유

은행 창구를 생각해보자.

고객 정보가 특정 직원 머릿속에만 있다면 그 직원이 쉬는 순간 업무가 멈춘다.

반대로 고객 상태가 중앙 시스템에 저장돼 있으면 어느 창구로 가도 업무를 처리할 수 있다.

Backend에서 Stateless Service가 이 두 번째 구조다.

## 2. Stateful vs Stateless

### Stateful
요청 처리에 필요한 상태를 특정 서버 메모리에 저장한다.

예:
```text
User A session → App Server 1 memory
```

다음 요청이 App Server 2로 가면 세션을 모를 수 있다.

### Stateless
요청 처리에 필요한 durable/shared state를 외부 저장소에 둔다.

예:
```text
Client
  ↓
Load Balancer
  ↓
Any App Server
  ↓
DB / Redis / Object Storage
```

## 3. 왜 중요한가

Stateless하면:
- horizontal scaling이 쉬워진다.
- instance 추가/제거가 쉬워진다.
- 장애 난 instance를 교체하기 쉽다.
- rolling deployment가 쉬워진다.
- load balancer가 요청을 자유롭게 분산할 수 있다.

## 4. 세션은 어디에 둘까

대표 선택:
- signed cookie/JWT처럼 client가 들고 다님
- Redis 같은 shared session store
- DB

각각 trade-off가 있다.

### JWT 장점
- server-side session lookup 감소
- 여러 instance에서 검증 가능

### JWT 주의
- revoke가 까다롭다.
- 너무 많은 정보를 넣으면 payload가 커진다.
- 민감 정보를 무작정 넣으면 안 된다.
- token lifetime 정책이 중요하다.

### Shared Session Store
Redis 등에 세션을 저장하면 revoke와 server-side control이 쉽지만 Redis availability와 network hop이 추가된다.

## 5. Sticky Session은 왜 덜 이상적인가

Sticky Session은 특정 client를 같은 server로 보내는 방식이다.

장점:
- 기존 stateful app을 빠르게 운영 가능

단점:
- load imbalance
- instance 장애 시 session 유실 위험
- scaling/deployment 유연성 감소

따라서 가능하면 app tier는 stateless하게 만드는 것이 일반적으로 유리하다.

## 6. Stateless가 '상태가 없다'는 뜻은 아니다

서비스 전체에는 상태가 당연히 존재한다.

핵심은:
> 상태를 특정 application instance의 로컬 메모리에 의존하지 않도록 하는 것

DB, cache, object storage, broker에는 state가 있다.

## 7. Local Cache는 써도 되나

쓸 수 있다. 다만 source of truth로 쓰면 안 된다.

예:
- immutable config
- short-lived cache
- compiled template

문제는 instance마다 값이 달라질 수 있다는 점이다.

따라서 consistency requirement를 이해해야 한다.

## 8. File Upload도 주의

사용자가 App Server 1의 local disk에 파일을 저장했는데 다음 요청이 App Server 2로 가면 파일이 없다.

그래서 보통:
- S3/Object Storage
- shared filesystem

등으로 분리한다.

## 9. Horizontal Scaling과 연결

Stateless service는 다음 구조와 잘 맞는다.

```text
Clients
  ↓
Load Balancer
  ↓
App 1  App 2  App 3
  ↓      ↓      ↓
Shared DB / Cache / Storage
```

App instance를 늘리기만 하면 처리량을 높이기 쉬워진다.

물론 DB 같은 downstream은 별도로 병목이 될 수 있다.

## 10. 장애 관점

Stateful instance가 죽으면:
- session
- in-memory job
- local file

같은 상태가 같이 사라질 수 있다.

Stateless 구조에서는 instance는 상대적으로 disposable하다.

이 개념은 cloud-native와 container orchestration의 핵심과도 연결된다.

## 11. 흔한 오해

### Stateless = DB를 안 쓴다
아니다. App instance가 요청 간 상태를 로컬에 붙잡지 않는다는 의미다.

### JWT를 쓰면 완벽한 stateless
아니다. authorization state, revoke policy, user data는 여전히 외부 state를 가질 수 있다.

### Sticky Session은 무조건 나쁘다
아니다. migration이나 특정 실시간 연결에서 현실적인 선택일 수 있지만 비용을 이해해야 한다.

## 12. 60초 면접 답변

> Stateless Service는 요청을 처리하는 데 필요한 durable state를 특정 application instance의 로컬 메모리에 의존하지 않는 구조입니다. 세션이나 사용자 상태는 Redis, DB, signed token 같은 외부 또는 공유 메커니즘에 두기 때문에 어느 instance로 요청이 가도 처리할 수 있습니다. 이렇게 하면 load balancer가 자유롭게 트래픽을 분산할 수 있고 horizontal scaling, rolling deployment, 장애 instance 교체가 쉬워집니다. 다만 stateless라고 시스템에 state가 없는 것은 아니며 DB, cache, object storage 같은 외부 state store의 availability와 consistency는 별도로 설계해야 합니다.
