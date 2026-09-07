# 20. Cache Consistency

## 한 줄 핵심

Cache는 DB 부하와 latency를 줄이지만, 데이터가 두 곳에 존재하면서 `어느 값이 최신인가`라는 정합성 문제가 생긴다.

## Cache-Aside

가장 흔한 패턴:

```text
Read
1. Cache 조회
2. Miss면 DB 조회
3. Cache 저장
4. 응답
```

Write는 보통:

```text
1. DB 변경
2. Cache 삭제/갱신
```

형태로 설계한다.

## 왜 DB 먼저 쓰고 Cache를 지우는가

예를 들어 Cache를 먼저 삭제한 뒤 DB Update가 느린 동안 다른 Request가 old DB 값을 읽어 Cache를 다시 채울 수 있다.

물론 어떤 순서도 완벽한 분산 Atomicity를 보장하지는 않는다. 중요한 것은 실패 시나리오를 명시적으로 생각하는 것이다.

## Update Cache vs Invalidate Cache

### Cache Update

DB 변경 후 새 값을 Cache에도 쓴다.

장점:
- 다음 Read가 바로 빠름

단점:
- DB와 Cache 두 곳의 Write 정합성 문제
- 어떤 표현/파생 Cache를 모두 갱신할지 복잡

### Cache Invalidation

DB 변경 후 관련 Key를 삭제한다.

장점:
- 다음 Read가 DB에서 최신 값을 다시 채움
- 구현이 비교적 단순

단점:
- 다음 Read는 Miss
- invalidation 누락 시 stale data

## TTL

TTL은 stale data가 영원히 남는 것을 제한하는 안전장치다.

하지만 TTL만 짧게 하면:

- Cache Miss 증가
- DB load 증가
- Stampede 가능성 증가

하므로 정합성과 부하의 trade-off가 있다.

## Cache Stampede

인기 Key가 동시에 만료되면 수많은 Request가 같은 DB Query를 수행할 수 있다.

대응:

- Single-flight / Request Coalescing
- Lock/Lease
- TTL Jitter
- Stale-while-revalidate

## Cache Penetration

존재하지 않는 Key를 계속 요청해서 매번 DB까지 가는 문제.

대응:

- Negative Caching
- 입력 검증
- 필요하면 Bloom Filter 같은 구조 고려

## Cache Avalanche

많은 Key가 비슷한 시점에 동시에 만료되어 DB 부하가 급증하는 현상.

TTL에 Jitter를 섞어 만료 시점을 분산할 수 있다.

## Read-After-Write 문제

사용자가 Profile을 수정한 직후 Cache에서 old value를 읽으면 UX 문제가 된다.

선택지:

- 해당 Key 즉시 invalidate
- 일정 시간 Primary/DB 직접 read
- versioned key
- write-through 계열 전략

요구되는 Consistency 수준에 따라 설계한다.

## Distributed Cache와 Local Cache

Local in-memory cache는 빠르지만 Instance마다 값이 달라질 수 있다.

```text
App A cache = new
App B cache = old
```

Distributed Cache는 공유 상태를 만들기 쉽지만 Network hop과 별도 장애 지점이 생긴다.

## Cache Key 설계

잘못된 Key는 성능 문제보다 더 위험할 수 있다.

사용자별 결과인데:

```text
/profile
```

만 Key로 쓰고 user identity를 포함하지 않으면 다른 사용자의 데이터를 반환할 위험이 있다.

따라서 Cache Key에는 응답을 실제로 결정하는 context가 반영되어야 한다.

## DB Transaction과 Cache

Local DB Transaction과 Redis 같은 외부 Cache는 일반적으로 하나의 Atomic Transaction이 아니다.

```text
DB COMMIT 성공
Cache delete 실패
```

같은 partial failure가 가능하다.

중요한 데이터에서는:

- Retry
- Outbox/Event 기반 invalidation
- TTL
- Reconciliation

등으로 eventual consistency를 관리한다.

## Cache는 Source of Truth인가?

대부분의 Cache-Aside 구조에서는 DB가 Source of Truth이고 Cache는 재생성 가능한 복사본이다.

이 원칙이 있으면 장애 시 Cache를 비우고 DB에서 복구하기 쉽다.

## 관찰해야 할 지표

- Hit Ratio
- Miss latency
- Eviction rate
- Stale-data incident
- DB QPS after miss
- Hot key
- Memory usage

Hit Ratio만 높다고 좋은 Cache는 아니다. 잘못된 stale data를 빠르게 반환할 수도 있기 때문이다.

## 60초 면접 답변

Cache는 DB latency와 부하를 줄이지만 같은 데이터가 DB와 Cache에 존재해서 consistency 문제가 생깁니다. 대표적으로 Cache-Aside에서는 Read 시 Cache miss면 DB에서 읽어 채우고, Write 시 DB를 먼저 변경한 뒤 Cache를 invalidate하는 방식을 많이 사용합니다. 다만 DB와 외부 Cache는 하나의 Atomic Transaction이 아니기 때문에 invalidation 실패와 stale data 가능성은 남습니다. 그래서 TTL, retry, event/outbox 기반 invalidation 같은 보완책을 사용합니다. 또한 인기 Key 만료 시 Cache Stampede가 생길 수 있어 single-flight, TTL jitter, stale-while-revalidate 등을 고려합니다. 중요한 것은 Cache를 빠른 저장소가 아니라 consistency trade-off가 있는 복제된 상태로 보는 것입니다.
