# 14. Replication and Read Replicas

## 한 줄 설명

Replication은 **한 Database의 변경 내용을 다른 Database Node로 복제해 가용성, 장애 대응, 읽기 확장성을 높이는 기술**입니다.

가장 흔한 구조는:

```text
Primary
  ├─ Replica A
  └─ Replica B
```

입니다.

Primary가 Write를 받고 Replica가 변경 내용을 따라갑니다.

---

## 왜 필요한가

단일 DB만 있으면:

- 장애 시 서비스 중단
- 읽기 트래픽이 몰리면 Primary 과부하
- 유지보수/Failover가 어려움

Replication을 사용하면:

- Read Scaling
- Failover 후보 확보
- 일부 지역/워크로드 분산

이 가능합니다.

하지만 복제 자체가 새로운 정합성 문제를 만듭니다.

---

## Synchronous vs Asynchronous Replication

### Synchronous

Primary Commit이 성공하기 전에 Replica의 확인까지 기다립니다.

장점:

- Replica가 더 최신 상태일 가능성이 높음
- Failover 시 데이터 손실 위험 감소

비용:

- Write Latency 증가
- 느린 Replica가 Primary 성능에 영향
- 네트워크 지연에 민감

### Asynchronous

Primary가 먼저 Commit하고 Replica는 나중에 따라갑니다.

장점:

- Write Latency가 상대적으로 낮음
- Primary가 Replica 지연에 덜 묶임

비용:

- Replication Lag
- Failover 시 최신 일부 Write 유실 가능성
- Read-after-write 문제

---

## Replication Lag

예:

```text
1. User가 Profile 이름 변경
2. Primary COMMIT 성공
3. 다음 GET 요청이 Replica로 감
4. Replica가 아직 이전 값을 반환
```

사용자는 방금 저장했는데 옛 값이 보입니다.

이것이 대표적인 **Read-after-write Consistency 문제**입니다.

---

## Read-after-write 대응

가능한 전략:

- 변경 직후 일정 시간 Primary에서 읽기
- 같은 Session/User의 후속 Read를 Primary로 Routing
- Replica가 특정 Log Position까지 따라올 때까지 대기
- Lag 허용 가능한 데이터만 Replica에서 읽기

핵심:

> 모든 Read를 Replica로 보내는 것은 단순한 성능 최적화가 아니라 consistency policy를 선택하는 일이다.

---

## 어떤 데이터가 Replica Read에 잘 맞는가

상대적으로 Lag 허용이 쉬운 것:

- 상품 Catalog
- 게시글 목록
- 통계
- 검색용 보조 데이터

Lag에 민감한 것:

- 결제 직후 상태
- 재고
- 계정 권한 변경 직후
- 비밀번호/보안 설정
- 방금 생성한 주문 상세

---

## Failover

Primary가 죽으면 Replica를 새 Primary로 승격할 수 있습니다.

하지만 Failover는 단순히 IP만 바꾸는 일이 아닙니다.

고려할 것:

- 어떤 Replica가 가장 최신인가
- 이전 Primary가 다시 살아났을 때 어떻게 할 것인가
- Client Connection 재연결
- DNS/Proxy Routing
- Split-brain 방지

---

## Split-brain

두 Node가 동시에 자신이 Primary라고 생각해 각각 Write를 받으면 데이터가 갈라질 수 있습니다.

이를 막기 위해 Election, Quorum, Fencing 같은 메커니즘이 필요할 수 있습니다.

면접에서는 세부 합의 알고리즘을 깊게 설명하지 못해도:

> Failover에서 가장 위험한 것은 두 Primary가 동시에 Write를 받는 상황이다.

를 이해해야 합니다.

---

## Replica는 Backup이 아니다

중요합니다.

```text
DROP TABLE users;
```

가 Primary에서 실행되면 그 변경도 Replica로 복제될 수 있습니다.

따라서 Replica는:

- Hardware Failure
- Node Failure

에는 도움되지만,

- 실수
- 잘못된 Migration
- 논리적 데이터 손상

까지 자동으로 보호하지는 않습니다.

별도 Backup과 Point-in-time Recovery 전략이 필요합니다.

---

## Read Scaling의 함정

Replica를 늘리면 Read capacity는 늘릴 수 있지만:

- Write bottleneck은 Primary에 남음
- Replica Lag가 증가할 수 있음
- Connection 수가 늘 수 있음
- Cache invalidation/Consistency가 복잡해짐

즉 Replica는 Sharding을 대신하지 않습니다.

---

## Connection Pool과 연결

Application Instance 20개가 있고 각 Instance가:

```text
Primary Pool 30
Replica Pool 30
```

을 가지면 최대 연결 수는 크게 증가할 수 있습니다.

```text
20 × (30 + 30) = 1200 connections
```

Replication 구조를 추가할 때 Connection Budget도 같이 봐야 합니다.

---

## 흔한 오해

### 오해 1. Replica에서 읽으면 항상 최신이다

Async Replication에서는 아닙니다.

### 오해 2. Replica가 있으면 Backup은 필요 없다

아닙니다. 논리적 삭제/오류도 복제될 수 있습니다.

### 오해 3. Replica를 늘리면 Write도 빨라진다

일반적인 Primary-Replica 구조에서는 Write bottleneck은 Primary에 남습니다.

---

## 면접 60초 답변

> Replication은 Primary의 변경 내용을 Replica로 복제해 가용성과 읽기 확장성을 높이는 기술입니다. Synchronous Replication은 Replica 확인을 Commit 경로에 포함해 데이터 손실 위험을 줄이는 대신 latency가 증가하고, Asynchronous Replication은 빠르지만 Replication Lag 때문에 stale read가 발생할 수 있습니다. 그래서 Read Replica를 사용할 때는 read-after-write consistency가 필요한 요청을 Primary로 보내는 등의 정책이 필요합니다. Failover 시에는 가장 최신 Replica를 승격하고 split-brain을 막아야 하며, Replica는 잘못된 DELETE 같은 논리적 오류도 복제할 수 있으므로 Backup을 대체하지 않습니다.
