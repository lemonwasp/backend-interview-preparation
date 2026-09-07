# 11. Service Boundary / Ownership

## 한 줄 정의

서비스 경계(Service Boundary)는 **어떤 데이터와 비즈니스 규칙을 어느 서비스가 책임지는지**를 나누는 기준이고, Ownership은 그 경계 안의 변경 권한과 책임을 명확히 하는 것이다.

---

## 1. 쉬운 비유

회사를 생각해보자.

- 인사팀은 급여와 인사 기록을 관리한다.
- 재무팀은 회계와 지급을 관리한다.
- 물류팀은 출고와 배송 상태를 관리한다.

모든 팀이 하나의 Excel 파일을 직접 수정한다면 처음에는 편하다.
하지만 조직이 커지면 누가 어떤 값을 바꿨는지, 어떤 규칙이 맞는지 알기 어려워진다.

서비스 경계도 비슷하다.

> "이 데이터의 최종 책임자는 누구인가?"

이 질문에 답할 수 있어야 한다.

---

## 2. 왜 서비스 경계가 필요한가

서비스를 분리하는 목적은 단순히 Repository를 여러 개로 만드는 것이 아니다.

좋은 경계는 다음을 줄인다.

- 서로 다른 팀의 동시 변경 충돌
- 하나의 기능 수정이 전체 시스템에 퍼지는 영향
- Shared Database로 인한 강한 결합
- 배포 시 전체 시스템을 같이 움직여야 하는 문제
- 서로 다른 확장 요구사항을 한 덩어리로 처리하는 문제

반대로 잘못 나누면 다음이 생긴다.

- 서비스 간 Network Call 폭증
- Distributed Transaction
- 데이터 중복/동기화
- 장애 전파
- Debugging 복잡도
- 팀 간 조율 비용 증가

즉, **서비스가 많다고 좋은 아키텍처가 아니다.**

---

## 3. 가장 중요한 기준: Business Capability

보통 좋은 경계는 기술 계층보다 비즈니스 능력에 가깝다.

나쁜 예:

```text
UserControllerService
UserRepositoryService
UserValidationService
```

이렇게 기술 계층별로 네트워크 서비스를 나누면 하나의 요청이 여러 서비스로 쪼개진다.

더 자연스러운 예:

```text
Identity Service
Order Service
Payment Service
Inventory Service
Shipping Service
```

각 서비스가 하나의 비즈니스 책임을 가진다.

---

## 4. Data Ownership

서비스 경계를 설계할 때 가장 강력한 질문은 이것이다.

> 이 데이터의 Source of Truth는 누구인가?

예를 들어 Order Service가 주문 상태의 Owner라면 다른 서비스는 주문 테이블을 직접 수정하지 않는 것이 기본이다.

```text
Payment Service
    ↓ event/API
Order Service
    ↓
orders.status 변경
```

Payment Service가 `orders` 테이블을 직접 UPDATE하면 서비스 경계는 사실상 무너진다.

---

## 5. Shared Database의 장단점

초기 시스템에서 Shared DB는 실용적일 수 있다.

장점:

- 구현 단순
- JOIN 쉬움
- Local Transaction 가능
- 운영 도구 단순

단점:

- Schema coupling
- 다른 팀이 내 테이블을 직접 수정
- 독립 배포 어려움
- DB가 모든 서비스의 공통 장애 지점
- 서비스 경계가 코드 레벨에만 존재할 수 있음

따라서 Shared DB 자체가 무조건 잘못은 아니지만, **누가 어떤 테이블을 소유하는지**는 명확해야 한다.

---

## 6. Database per Service

강한 독립성이 필요하면 서비스별 저장소를 분리할 수 있다.

```text
Order Service -> Order DB
Payment Service -> Payment DB
Inventory Service -> Inventory DB
```

장점:

- 독립 Schema Evolution
- 독립 확장
- 데이터 변경 책임 명확

비용:

- Cross-service JOIN 불가
- Distributed Consistency 필요
- 데이터 복제/Projection 필요
- Saga/Outbox/Event 설계 필요

즉 Database per Service는 무료 독립성이 아니다.

---

## 7. 동기 API vs 비동기 Event

서비스 간 통신 방식도 경계의 일부다.

### 동기 API

```text
Order -> Payment -> Inventory
```

장점:
- 즉각적인 결과
- 흐름이 이해하기 쉬움

단점:
- latency 합산
- downstream 장애가 upstream으로 전파
- 긴 dependency chain

### 비동기 Event

```text
OrderCreated
   ↓
Payment
Inventory
Analytics
```

장점:
- 결합도 감소
- spike 흡수
- 독립 처리

단점:
- eventual consistency
- duplicate/order/retry
- debugging 어려움

---

## 8. 서비스 경계와 Transaction Boundary

가장 중요한 원칙 중 하나:

> 강한 Transaction이 자주 필요한 데이터는 같은 경계 안에 둘 이유가 있다.

예를 들어 주문 생성과 주문 라인 저장이 항상 하나의 atomic transaction이어야 한다면 이를 무리하게 서비스로 쪼개면 distributed transaction 문제가 생긴다.

반대로 결제 승인과 배송 시작은 비동기적으로 연결될 수 있다.

---

## 9. Microservice를 너무 일찍 나누면

작은 팀에서 도메인이 아직 불안정한데 Microservice를 많이 만들면 다음 문제가 커진다.

- 인터페이스 변경 비용
- 배포/모니터링 비용
- CI/CD 수 증가
- observability 필요성 증가
- 개발 환경 복잡도
- local transaction 상실

그래서 초기에는 **Modular Monolith**가 더 나을 수 있다.

```text
One Deployable
├── Order Module
├── Payment Module
└── Inventory Module
```

모듈 간 경계를 코드로 강하게 유지하고, 필요해질 때 서비스로 분리한다.

---

## 10. 언제 분리할까

분리 신호의 예:

- 팀 Ownership이 명확히 갈림
- 배포 주기가 크게 다름
- 확장 패턴이 크게 다름
- 장애 격리가 필요함
- 보안/규제 경계가 다름
- 독립적인 데이터 수명 주기가 있음

반대로 단순히 "Microservice가 최신이니까"는 이유가 아니다.

---

## 11. Backend 면접 연결

면접에서 "서비스를 어떻게 나누시겠어요?"라고 물으면 아래 순서가 좋다.

1. 핵심 Business Capability를 찾는다.
2. Source of Truth와 Data Owner를 정한다.
3. Strong Transaction이 필요한 범위를 확인한다.
4. Sync/Async interaction을 선택한다.
5. 장애 전파와 운영 비용을 본다.
6. 팀 규모와 배포 독립성이 정말 필요한지 확인한다.

---

## 12. 흔한 오해

### 오해 1: Microservice가 Monolith보다 항상 좋다

아니다. 분산 시스템의 복잡성을 지불할 가치가 있을 때만 이점이 있다.

### 오해 2: 서비스마다 무조건 DB가 따로 있어야 한다

궁극적인 독립성에는 도움이 되지만 모든 조직/단계에서 최선은 아니다.

### 오해 3: Event-driven이면 결합이 사라진다

아니다. Event Schema와 의미에 대한 결합은 여전히 존재한다.

### 오해 4: 서비스 경계는 코드 패키지 경계와 같다

아니다. 데이터 Ownership, 배포, 장애, Transaction Boundary까지 포함한다.

---

## 13. 60초 기술면접 답변

> 서비스 경계는 비즈니스 책임과 데이터 Ownership을 기준으로 나누는 것이 핵심입니다. 먼저 어떤 서비스가 특정 데이터의 Source of Truth인지 정하고, 강한 Transaction이 필요한 데이터는 같은 경계에 유지할 이유가 있는지 봅니다. 서비스 간 통신은 즉시 결과가 필요하면 동기 API를, 느슨한 결합과 비동기 처리가 중요하면 Event를 고려합니다. Microservice는 독립 배포와 장애 격리 장점이 있지만 Network Call, Distributed Consistency, Observability 비용도 생기므로 작은 팀이나 불안정한 도메인에서는 Modular Monolith가 더 적절할 수 있습니다.
