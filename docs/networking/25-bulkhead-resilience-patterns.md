# 25. Bulkhead and Resilience Patterns

## Bulkhead란

배의 격벽(Bulkhead)은 한 구역에 물이 들어와도 배 전체가 침수되지 않게 구역을 나눕니다.

소프트웨어에서도 같은 생각을 사용합니다.

하나의 느리거나 장애 난 dependency가 모든 Thread, Connection, Memory를 먹어버리지 못하도록 자원을 분리합니다.

## 예시

Service가 두 외부 API를 호출한다고 가정합니다.

```text
Payment API
Recommendation API
```

Recommendation API가 느려졌다고 모든 Worker Thread가 여기에 묶이면 Payment까지 느려질 수 있습니다.

그래서 각각 별도의 concurrency limit, thread pool, connection pool 또는 queue를 둘 수 있습니다.

```text
Payment -> max 50 concurrent
Recommendation -> max 10 concurrent
```

Recommendation 장애가 전체 서버를 먹지 못하게 하는 것입니다.

## Bulkhead 구현 방법

- 별도 Thread Pool
- 별도 Connection Pool
- Semaphore / concurrency limit
- 별도 Queue
- Process / Container 분리
- Service 자체 분리

격리 수준이 강해질수록 보호는 좋아지지만 자원 활용률과 운영 복잡도도 증가할 수 있습니다.

## Semaphore Bulkhead

예를 들어 Recommendation 호출을 동시에 최대 20개만 허용합니다.

21번째 요청은 기다리거나 빠르게 실패합니다.

이렇게 하면 느린 dependency가 전체 request worker를 끝없이 점유하는 것을 막을 수 있습니다.

## Queue도 무한하면 안 된다

Concurrency를 20으로 제한했는데 Queue가 무한이라면 결국 Memory와 Latency가 폭증할 수 있습니다.

따라서:

```text
bounded concurrency
+ bounded queue
+ timeout
```

을 함께 보는 것이 중요합니다.

## Backpressure

생산 속도가 소비 속도보다 빠르면 시스템은 밀립니다.

Backpressure는 downstream이 감당할 수 없는 속도를 upstream이 계속 밀어 넣지 않도록 제어하는 개념입니다.

방법:

- 요청 거부
- queue size 제한
- producer 속도 감소
- streaming flow control
- 429 / 503 반환

## Load Shedding

시스템이 모든 요청을 처리하다 같이 죽는 것보다 일부 요청을 의도적으로 버려 핵심 기능을 보호하는 전략입니다.

예:

```text
로그 분석 기능 거부
추천 기능 비활성화
핵심 결제 기능 유지
```

## Graceful Degradation

전체 기능이 아니더라도 핵심 기능만 제공하는 방식입니다.

예:

- 추천 서버 장애 -> 인기 상품 목록 반환
- 이미지 서버 장애 -> placeholder 표시
- 실시간 통계 장애 -> 마지막 cache 값 반환

## Resilience Pattern을 조합하기

실전에서는 하나만 쓰지 않습니다.

```text
Rate Limit
→ Timeout
→ Retry + Backoff + Jitter
→ Circuit Breaker
→ Bulkhead
→ Load Shedding / Fallback
```

각 패턴은 다른 문제를 해결합니다.

| Pattern | 주요 목적 |
|---|---|
| Rate Limit | 유입량 제한 |
| Timeout | 오래 기다리지 않기 |
| Retry | 일시 실패 복구 |
| Circuit Breaker | 반복 장애 빠르게 격리 |
| Bulkhead | 자원 고갈 범위 제한 |
| Backpressure | 생산/소비 속도 조정 |
| Load Shedding | 과부하 시 일부 작업 포기 |

## 실패 예시

### Thread Pool Starvation

느린 dependency 때문에 모든 Thread가 blocking되면 정상 dependency 호출도 못 합니다.

Bulkhead로 전용 pool이나 concurrency limit을 두면 피해 범위를 제한할 수 있습니다.

### Connection Pool Exhaustion

하나의 DB/API가 connection을 오래 잡으면 전체 pool이 고갈될 수 있습니다.

Dependency별 pool과 timeout이 도움이 됩니다.

## Resilience vs Availability

Resilience는 "절대 실패하지 않는다"가 아닙니다.

장애가 발생해도:
- 피해 범위를 줄이고
- 핵심 기능을 유지하고
- 회복할 수 있도록 만드는 능력입니다.

## Backend 면접 연결

질문: "추천 서비스 장애 때문에 전체 쇼핑몰 API가 느려졌습니다. 어떻게 막겠습니까?"

답변 예:

```text
추천 호출 concurrency limit
→ 짧은 timeout
→ circuit breaker
→ fallback 인기 상품
→ 추천용 자원과 결제용 자원 격리
```

## 흔한 오해

### "Bulkhead는 Microservice로 나누는 것이다"

Microservice 분리는 한 방법일 뿐입니다. 같은 Process 안에서도 Semaphore나 Thread Pool 분리로 Bulkhead를 만들 수 있습니다.

### "Queue를 두면 과부하 문제는 해결된다"

무한 Queue는 장애를 숨기다가 더 큰 latency와 memory 문제로 바꿀 수 있습니다.

## 60초 면접 답변

Bulkhead는 한 dependency의 장애나 지연이 시스템 전체 자원을 고갈시키지 않도록 Thread Pool, Connection Pool, Semaphore, Queue 같은 자원을 격리하는 resilience pattern입니다. 예를 들어 추천 API 호출 동시성을 10개로 제한하면 추천 API가 느려져도 결제 기능까지 모든 Thread를 빼앗기지 않게 할 수 있습니다. Bulkhead는 bounded queue와 timeout, circuit breaker, rate limiting, backpressure 같은 패턴과 함께 사용합니다. 핵심은 모든 요청을 끝까지 처리하는 것이 아니라 장애 시 피해 범위를 제한하고 핵심 기능을 유지하는 것입니다.