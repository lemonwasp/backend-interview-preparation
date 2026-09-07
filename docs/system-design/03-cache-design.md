# 03. Cache Design

Cache의 목적은 단순히 "빠르게 만들기"가 아니라 **비싼 원본 조회를 반복하지 않도록 해서 latency와 downstream 부하를 줄이는 것**이다.

## 1. 비유

자주 보는 책을 매번 창고에서 꺼내오지 않고 책상 위에 두는 것과 같다.

- 책상: 빠르지만 공간이 작음
- 창고: 느리지만 원본이 있음

Backend에서는:
- Cache: Redis, local memory, CDN
- Source of Truth: DB, Object Storage, external service

## 2. 언제 Cache가 효과적인가

특히 다음 조건에서 효과가 크다.
- 읽기가 많다.
- 같은 데이터가 반복 조회된다.
- 원본 조회가 비싸다.
- 데이터가 조금 늦게 반영돼도 된다.

반대로 데이터가 매번 바뀌거나 강한 정합성이 필수라면 cache가 복잡도를 키울 수 있다.

## 3. Cache-Aside

가장 대표적인 패턴이다.

```text
1. Cache 조회
2. Hit → 바로 반환
3. Miss → DB 조회
4. Cache 저장
5. 반환
```

Write 시 보통:

```text
DB update
→ cache invalidate
```

이 방식은 단순하지만 DB commit과 cache invalidation은 하나의 atomic transaction이 아니다.

## 4. Write-Through / Write-Behind

### Write-Through
쓰기 요청을 cache와 backing store에 함께 반영한다.

장점:
- cache가 비교적 최신 상태 유지

단점:
- write latency 증가
- 구현 복잡도 증가

### Write-Behind
먼저 cache에 쓰고 뒤에서 DB에 반영한다.

장점:
- write latency 감소
- batching 가능

단점:
- cache 장애 시 데이터 유실 위험
- durability/ordering 설계가 복잡

## 5. TTL

Cache는 영원히 두지 않고 TTL을 둔다.

TTL이 짧으면:
- stale data 감소
- DB load 증가

TTL이 길면:
- hit ratio 증가
- stale data 증가

따라서 데이터 성격에 맞춰야 한다.

## 6. Cache Hit Ratio

```text
Hit Ratio = Cache Hit / Total Cache Request
```

Hit Ratio가 높을수록 DB load 감소 효과가 크다.

하지만 Hit Ratio 하나만 보면 안 된다.
- latency
- cache memory
- eviction
- DB query rate
- hot key

도 같이 봐야 한다.

## 7. Cache Stampede

인기 key가 동시에 만료되면 수많은 요청이 DB로 몰릴 수 있다.

```text
Hot Key expires
→ 10,000 requests miss
→ 10,000 DB queries
```

완화 방법:
- single-flight/request coalescing
- TTL jitter
- stale-while-revalidate
- background refresh

## 8. Cache Penetration

존재하지 않는 데이터를 반복 조회해 cache miss가 계속 나는 경우다.

완화:
- negative caching
- input validation
- Bloom Filter 같은 기법

단, negative cache TTL을 너무 길게 두면 새로 생성된 데이터가 늦게 보일 수 있다.

## 9. Cache Avalanche

많은 key가 비슷한 시각에 동시에 만료되는 현상이다.

완화:
- TTL jitter
- expiration 분산
- capacity 보호

## 10. Hot Key

특정 key 하나에 트래픽이 몰리는 문제다.

예:
- 인기 게시물
- 실시간 랭킹
- 유명인의 프로필

분산 cache라도 한 shard/node가 집중될 수 있다.

## 11. Local Cache vs Distributed Cache

### Local Cache
장점:
- 매우 빠름
- network hop 없음

단점:
- instance별 값 불일치
- 메모리 중복
- invalidation 어려움

### Distributed Cache
장점:
- 여러 app instance가 공유
- 일관된 cache state 관리 쉬움

단점:
- network hop
- cache 자체가 dependency
- cluster 운영 필요

## 12. Cache Key Design

좋은 key는:
- 충분히 구체적
- collision 없음
- tenant/user isolation 고려
- versioning 가능

예:

```text
user:{userId}:profile:v2
```

잘못된 tenant key 설계는 데이터 유출로 이어질 수 있다.

## 13. Eviction Policy

메모리가 꽉 차면 어떤 데이터를 버릴지 정해야 한다.

대표:
- LRU 계열
- LFU 계열
- TTL 기반

"Redis를 쓴다"보다 **어떤 데이터가 어떤 eviction policy 아래 살아남아야 하는가**가 더 중요하다.

## 14. Cache와 Consistency

Cache는 source of truth가 아니다.

문제:
```text
DB update 성공
cache invalidation 실패
→ stale data
```

완화:
- TTL
- retry
- outbox/event
- versioned key
- reconciliation

Database에서 배운 Cache Consistency와 직접 연결된다.

## 15. CDN도 Cache다

정적 파일, 이미지, 큰 다운로드는 origin app까지 보내지 않고 edge에서 처리할 수 있다.

효과:
- latency 감소
- origin bandwidth 감소
- app server 부하 감소

## 16. 언제 Cache를 넣지 말까

다음이면 신중해야 한다.
- 트래픽이 아직 작음
- DB query가 충분히 빠름
- 데이터가 매우 자주 바뀜
- correctness가 cache 복잡도보다 중요

Cache는 무료 성능이 아니라 **일관성 복잡도를 대가로 사는 성능**이다.

## 17. 60초 면접 답변

> Cache는 반복되는 비싼 원본 조회를 줄여 latency와 downstream 부하를 낮추는 계층입니다. 가장 흔한 방식은 Cache-Aside로, 먼저 cache를 보고 miss면 DB에서 읽어 cache에 채웁니다. 다만 cache는 source of truth가 아니기 때문에 stale data, invalidation failure, stampede, hot key 같은 문제가 생깁니다. 그래서 TTL, single-flight, TTL jitter, versioned key 같은 전략을 함께 설계해야 합니다. 또 local cache는 빠르지만 instance 간 불일치가 있고 distributed cache는 공유가 쉽지만 network와 운영 비용이 추가됩니다. Cache는 트래픽과 consistency 요구사항을 보고 선택해야 합니다.
