# 27. Backpressure Deep Dive

## 한 줄 요약

Backpressure는 Producer가 Consumer보다 빠를 때, 시스템이 무한히 쌓이지 않도록 **생산 속도를 늦추거나 일부 요청을 거절하는 압력 전달 메커니즘**이다.

## 가장 쉬운 비유

식당 주방이 분당 10접시만 만들 수 있는데 주문이 분당 100개 들어오면 어떻게 될까?

주문을 무한정 받으면 대기표만 늘어난다.

컴퓨터 시스템도 같다.

```text
Producer 1000 req/s
        ↓
Consumer 200 req/s
```

차이인 800 req/s가 계속 Queue에 쌓이면 결국:

- Latency 증가
- Memory 증가
- Timeout 증가
- GC pressure 증가
- 장애 확산

이 발생한다.

## Backpressure가 없는 시스템

```text
Traffic 증가
-> Queue 증가
-> 응답 지연
-> Client Timeout
-> Retry 증가
-> Queue 더 증가
-> 장애
```

이것은 앞서 배운 Retry Amplification과도 연결된다.

## 핵심 아이디어

Consumer가 감당할 수 없다는 신호를 Producer 쪽으로 전달해야 한다.

방법은 여러 가지다.

### 1. Bounded Queue

Queue 크기에 상한을 둔다.

```text
capacity = 1000
```

1000개가 찼다면 더 이상 무조건 받지 않는다.

선택지는:

- Block
- Reject
- Drop
- Shed load

등이다.

### 2. Semaphore / Concurrency Limit

동시에 수행할 작업 수를 제한한다.

```text
max concurrent requests = 100
```

이렇게 하면 DB Connection이나 Thread가 무한정 소모되지 않는다.

### 3. Rate Limiting

입력 자체를 줄인다.

Backpressure와 Rate Limiting은 관련 있지만 완전히 같은 개념은 아니다.

- Rate Limiting: 정책 기반 유입 제한
- Backpressure: 실제 소비 능력에 맞춰 upstream에 압력 전달

### 4. Reactive Stream Demand

Consumer가 "나는 지금 N개 받을 수 있다"는 Demand를 명시할 수 있다.

개념적으로:

```text
Consumer -> Producer: give me 100 items
```

Producer가 Consumer capacity를 무시하지 않는 구조다.

## Blocking은 Backpressure인가

경우에 따라 그렇다.

Producer Thread를 막아서 생산 속도를 늦추는 것도 Backpressure의 한 형태가 될 수 있다.

하지만 무작정 많은 Thread를 Blocking시키면 Thread Pool 고갈이 생길 수 있다.

따라서 async system에서는 bounded queue, semaphore, demand signaling이 더 적합할 수 있다.

## Load Shedding

시스템이 살기 위해 일부 요청을 버리는 전략이다.

예:

```text
현재 서버 CPU 98%
Queue full
-> 중요하지 않은 추천 API 요청은 503
-> 결제 API 자원은 보호
```

모든 요청을 받아서 모두 실패하는 것보다 일부를 빠르게 거절해 핵심 기능을 살리는 편이 낫다.

## Backpressure vs Buffering

Buffer는 순간적인 Burst를 흡수할 수 있다.

하지만 Producer 평균 속도가 Consumer 평균 속도보다 계속 빠르면 Buffer는 해결책이 아니다.

```text
평균 입력 1000/s
평균 처리 200/s
```

Buffer 크기를 10배 늘려도 장애 시점만 늦출 뿐이다.

## Message Queue에서의 Backpressure

Kafka/RabbitMQ 같은 Queue를 쓴다고 Backpressure 문제가 사라지는 것은 아니다.

봐야 할 것:

- Consumer Lag
- Queue Depth
- Processing Rate
- Retry Queue
- DLQ

Lag이 계속 증가하면 Consumer capacity가 부족하다는 뜻이다.

## Database 연결

DB Connection Pool이 50개인데 App Worker가 500개 동시에 Query를 보내면 대기열이 생긴다.

좋은 설계는:

- Connection Pool 크기 제한
- Request Concurrency 제한
- Queue 상한
- Timeout

을 함께 본다.

## TCP Flow Control과 연결

TCP의 Receiver Window도 넓은 의미에서는 Receiver capacity를 Sender에게 알려주는 흐름 제어다.

다만 Application-level Backpressure와 TCP Flow Control은 같은 층의 개념은 아니다.

## Observability

Backpressure를 운영하려면 다음 지표가 중요하다.

- Queue depth
- Queue wait time
- Consumer lag
- In-flight requests
- Rejection count
- Timeout rate
- p95 / p99 latency

## 흔한 오해

### "Queue만 넣으면 안전하다"

아니다. 무한 Queue는 OOM과 초장기 latency를 만든다.

### "더 많이 Buffering하면 해결된다"

평균 처리량 불균형은 Buffer로 해결되지 않는다.

### "503은 실패니까 무조건 나쁘다"

과부하 상황에서 빠른 503은 전체 시스템 붕괴보다 낫다.

## 60초 면접 답변

> Backpressure는 Producer의 처리 속도가 Consumer의 처리 능력을 초과할 때 무한 Queue와 자원 고갈을 막기 위해 upstream의 생산 속도를 조절하는 메커니즘입니다. 대표적으로 bounded queue, concurrency limit, semaphore, demand signaling, load shedding을 사용할 수 있습니다. Queue는 짧은 burst는 흡수하지만 평균 입력 속도가 처리 속도보다 계속 높다면 근본 해결책이 아닙니다. 실무에서는 queue depth, wait time, consumer lag, rejection, p99 latency를 보고 과부하를 감지하고 필요하면 일부 요청을 빠르게 거절해 핵심 기능을 보호합니다.
