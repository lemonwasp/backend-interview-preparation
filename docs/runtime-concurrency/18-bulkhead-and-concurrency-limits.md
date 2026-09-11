# Bulkhead Pattern and Concurrency Limits

## 1. 한 문장으로 설명

**Bulkhead Pattern은 한 기능의 과부하나 장애가 다른 기능까지 전파되지 않도록 자원 풀을 분리하는 장애 격리 패턴이고, Concurrency Limit은 동시에 실행되는 작업 수에 상한을 두는 기법이다.**

배의 격벽(bulkhead)을 떠올리면 쉽다.

```text
+---------+---------+---------+
| Section | Section | Section |
|    A    |    B    |    C    |
+---------+---------+---------+
```

한 구역에 물이 들어와도 다른 구역까지 바로 침수되지 않는다.

백엔드에서는 이를 다음처럼 적용한다.

```text
Payment      -> pool A
Email        -> pool B
Image Resize -> pool C
```

Image Resize가 폭주해도 Payment가 사용하는 스레드, 커넥션, 큐를 전부 소비하지 못하게 막는 것이다.

---

## 2. 왜 필요한가?

다음 서버가 있다고 하자.

```text
API Server
  |
  +-- Payment API
  +-- Email API
  +-- Image Processing API
```

세 기능이 모두 같은 Thread Pool과 DB Connection Pool을 공유한다면 문제가 생길 수 있다.

```text
Image Processing 폭주
        |
        v
Thread / CPU / Queue 고갈
        |
        v
Payment도 느려짐
        |
        v
전체 서비스 장애
```

즉 실제 장애 원인은 이미지 처리였는데 피해는 결제까지 번진다.

Bulkhead의 목적은 이것이다.

> **한 부분의 실패를 전체 실패로 만들지 않는다.**

---

## 3. Concurrency Limit이란?

Concurrency Limit은 동시에 처리하는 작업 개수의 최대값이다.

```text
Incoming requests: 1000

Concurrency Limit = 50

Running: 50
Waiting / Rejected: 950
```

중요한 점은 `requests per second`와 다르다는 것이다.

```text
Rate Limit
= 시간당 또는 초당 얼마나 많이 들어올 수 있는가?

Concurrency Limit
= 지금 이 순간 몇 개를 동시에 실행할 수 있는가?
```

예를 들어 요청 하나가 오래 걸리면 초당 요청 수가 많지 않아도 동시 실행 수는 크게 증가할 수 있다.

---

## 4. 왜 동시 실행 수를 제한해야 하나?

백엔드의 자원은 대부분 무한하지 않다.

예:

- CPU core
- Thread
- DB Connection
- Socket
- Memory
- External API quota
- File handle

예를 들어 DB Connection Pool이 100개인데 1,000개의 요청이 동시에 DB를 사용하려고 하면:

```text
100 connections in use
900 requests waiting
```

이때 요청을 무제한으로 받아들이면 다음 문제가 생긴다.

```text
Waiting requests ↑
Memory ↑
Timeout ↑
Retries ↑
Latency ↑
```

따라서 안정적인 시스템은 자원의 실제 처리 능력을 기준으로 동시성을 제한한다.

---

## 5. Semaphore로 만드는 간단한 Concurrency Limit

C#에서는 `SemaphoreSlim`을 이용해 동시에 실행되는 작업 수를 제한할 수 있다.

```csharp
private readonly SemaphoreSlim _limit = new(20);

public async Task HandleAsync(CancellationToken cancellationToken)
{
    await _limit.WaitAsync(cancellationToken);

    try
    {
        await ProcessAsync(cancellationToken);
    }
    finally
    {
        _limit.Release();
    }
}
```

여기서:

```csharp
new SemaphoreSlim(20)
```

은 동시에 최대 20개의 작업만 실행할 수 있다는 뜻이다.

21번째 요청부터는 permit이 반환될 때까지 기다린다.

구조는 다음과 같다.

```text
Requests
   |
   v
Semaphore(20)
   |
   +--> Running 20
   |
   +--> Waiting
```

---

## 6. 기다리기만 하면 충분할까?

아니다.

대기열까지 무한하다면 다시 unbounded queue 문제가 생긴다.

```text
Concurrency Limit = 20
Waiting Queue = unlimited
```

이 구조에서는 실행 중인 작업 수는 제한되지만 대기 요청은 계속 쌓일 수 있다.

따라서 실무에서는 보통 두 개를 함께 생각해야 한다.

```text
Concurrency Limit
+
Bounded Queue
```

예:

```text
Max running = 20
Max waiting = 100
```

101번째 대기 요청부터는 빠르게 실패시킬 수 있다.

---

## 7. Bulkhead의 핵심: 자원을 분리한다

나쁜 구조:

```text
                 +-------------------+
Payment -------->|                   |
Email ---------->| Shared ThreadPool |
Image Processing>|                   |
                 +-------------------+
```

어느 하나가 폭주하면 모두 영향을 받는다.

Bulkhead 구조:

```text
Payment --------> [ Pool A: 20 ]
Email ----------> [ Pool B: 10 ]
Image Processing> [ Pool C: 5  ]
```

이제 Image Processing이 폭주해도:

```text
Pool C = FULL
```

일 뿐이다.

Payment의 Pool A는 그대로 사용할 수 있다.

---

## 8. 무엇을 분리할 수 있는가?

Bulkhead는 Thread Pool만 분리하는 것이 아니다.

### 1) Worker / Thread

```text
Payment workers: 20
Email workers: 5
```

### 2) Queue

```text
Payment Queue
Email Queue
Notification Queue
```

### 3) DB Connection Pool

중요 업무와 비중요 업무가 동일한 DB 자원을 모두 소비하지 않도록 분리할 수 있다.

### 4) HTTP Client Connection

외부 서비스별로 connection limit과 timeout을 다르게 둘 수 있다.

### 5) Process / Container / Pod

더 강한 격리가 필요하면 애플리케이션 인스턴스 자체를 분리할 수 있다.

```text
Payment Pods
Email Pods
Image Pods
```

격리 수준이 높아질수록 장애 전파는 줄지만 운영 비용과 복잡성은 증가한다.

---

## 9. Bulkhead와 Microservice

Microservice Architecture도 어느 정도는 bulkhead와 비슷한 효과를 가진다.

```text
Monolith

Payment + Search + Recommendation
        |
   same process
```

하나의 프로세스가 죽으면 모든 기능이 영향을 받을 수 있다.

반면:

```text
Payment Service
Search Service
Recommendation Service
```

프로세스와 배포 단위가 분리되기 때문에 장애 격리가 쉬워질 수 있다.

하지만 중요한 점은:

> Microservice라고 자동으로 장애가 격리되는 것은 아니다.

모든 서비스가 같은 DB, 같은 Redis, 같은 Connection Pool 또는 같은 외부 API에 의존하면 여전히 공통 장애 지점이 생긴다.

---

## 10. Bulkhead와 Circuit Breaker의 차이

둘 다 resilience pattern이지만 목적이 다르다.

### Bulkhead

```text
"한 기능이 자원을 다 먹지 못하게 한다."
```

목적:

- 자원 격리
- 장애 전파 방지

### Circuit Breaker

```text
"이미 실패 중인 downstream 호출을 잠시 중단한다."
```

목적:

- 반복 실패 차단
- downstream 보호
- 빠른 실패

둘은 같이 사용하면 좋다.

```text
Request
  |
Bulkhead
  |
Circuit Breaker
  |
Downstream Service
```

---

## 11. Bulkhead와 Backpressure의 관계

이전 문서의 Backpressure와 연결해보자.

### Backpressure

```text
Consumer가 느려지면 Producer 속도를 줄인다.
```

### Concurrency Limit

```text
동시에 처리할 수 있는 작업 수를 제한한다.
```

### Bounded Queue

```text
대기 가능한 작업 수를 제한한다.
```

### Bulkhead

```text
기능별로 자원 자체를 분리한다.
```

결합하면:

```text
                +----------------------+
Payment ------> | concurrency = 20     |
                | queue = 100          |
                +----------------------+

                +----------------------+
Email --------> | concurrency = 5      |
                | queue = 200          |
                +----------------------+

                +----------------------+
Image --------> | concurrency = 3      |
                | queue = 20           |
                +----------------------+
```

각 기능은 서로 다른 failure domain을 가진다.

---

## 12. 실무 장애 시나리오

### 상황

Recommendation API가 외부 AI API를 호출한다.

```text
User Request
   |
   v
Recommendation API
   |
   v
External AI API
```

외부 AI API가 느려졌다.

Bulkhead가 없다면:

```text
AI API slow
   |
Request tasks accumulate
   |
Thread / socket / memory pressure
   |
Other endpoints become slow
```

Bulkhead를 적용하면:

```text
Recommendation concurrency = 30
Queue = 50
```

80개를 넘는 요청은 빠르게 실패하거나 fallback을 사용한다.

```text
Recommendation degraded

BUT

Login works
Payment works
Profile works
```

이것이 장애 격리다.

---

## 13. C#에서 Timeout까지 같이 적용하기

Concurrency Limit은 timeout과 같이 사용하는 것이 중요하다.

```csharp
private readonly SemaphoreSlim _limit = new(20);

public async Task<Result> CallExternalApiAsync(
    CancellationToken cancellationToken)
{
    using var timeoutCts = CancellationTokenSource.CreateLinkedTokenSource(
        cancellationToken);

    timeoutCts.CancelAfter(TimeSpan.FromSeconds(2));

    await _limit.WaitAsync(timeoutCts.Token);

    try
    {
        return await CallApiAsync(timeoutCts.Token);
    }
    finally
    {
        _limit.Release();
    }
}
```

중요한 이유:

```text
Concurrency = 20
```

이어도 각 작업이 영원히 끝나지 않는다면 20개의 permit이 계속 점유된다.

그래서:

```text
Concurrency Limit + Timeout
```

조합이 필요하다.

---

## 14. 한도를 어떻게 정할까?

정답은 하나가 없다.

다음을 측정해야 한다.

- 평균 latency
- p95 / p99 latency
- CPU 사용률
- DB Connection 사용률
- Queue length
- timeout rate
- downstream capacity

예를 들어:

```text
DB max connections = 100
```

이라고 해서 애플리케이션 동시성을 정확히 100으로 두는 것이 항상 옳지는 않다.

관리용 연결, 다른 서비스, transaction duration까지 고려해야 한다.

따라서 처음에는 보수적으로 설정하고 부하 테스트를 통해 조정한다.

---

## 15. 너무 낮게 잡으면?

Concurrency Limit은 높다고 무조건 나쁘고 낮다고 무조건 좋은 것이 아니다.

너무 낮으면:

```text
CPU idle
DB idle
Throughput low
```

너무 높으면:

```text
Queue ↑
Context Switching ↑
Memory ↑
DB contention ↑
Latency ↑
```

따라서 목표는:

> **자원을 포화시키되 붕괴시키지 않는 수준**

을 찾는 것이다.

---

## 16. 면접 질문

### Q1. Bulkhead Pattern이 무엇인가요?

> 선박의 격벽처럼 하나의 기능이나 downstream 장애가 전체 시스템으로 전파되지 않도록 Thread Pool, Queue, Connection Pool, Worker 등을 기능별로 분리하는 장애 격리 패턴입니다.

### Q2. Concurrency Limit과 Rate Limit의 차이는 무엇인가요?

> Rate Limit은 일정 시간 동안 들어오는 요청 수를 제한하고, Concurrency Limit은 특정 순간에 동시에 실행되는 작업 수를 제한합니다. 오래 걸리는 요청이 많은 시스템에서는 요청률이 낮아도 높은 동시성으로 자원이 고갈될 수 있기 때문에 둘은 별도로 관리해야 합니다.

### Q3. Semaphore만 사용하면 overload 문제가 해결되나요?

> 아닙니다. 실행 중인 작업 수는 제한되지만 대기 큐가 무한하면 요청이 계속 메모리에 쌓일 수 있습니다. 그래서 bounded queue, timeout, rejection policy와 함께 설계해야 합니다.

### Q4. 왜 모든 기능이 하나의 Thread Pool을 공유하면 위험할 수 있나요?

> 비중요 기능의 장시간 작업이나 폭주가 모든 worker를 점유하면 결제나 로그인 같은 중요 기능도 실행 기회를 얻지 못할 수 있기 때문입니다. Bulkhead를 적용하면 기능별 자원 한도를 분리해 이런 장애 전파를 줄일 수 있습니다.

### Q5. Bulkhead와 Circuit Breaker는 어떻게 다르나요?

> Bulkhead는 자원을 분리해 장애의 범위를 제한하는 패턴이고, Circuit Breaker는 실패 중인 downstream 호출 자체를 일정 시간 차단해 반복 실패와 자원 낭비를 막는 패턴입니다.

---

## 17. 60초 답변

> Bulkhead Pattern은 한 기능의 과부하나 장애가 다른 기능까지 전파되지 않도록 Thread Pool, Queue, Connection Pool 같은 자원을 기능별로 분리하는 패턴입니다. 여기에 Concurrency Limit을 사용하면 각 기능이 동시에 실행할 수 있는 작업 수에 상한을 둘 수 있습니다. 다만 동시 실행 수만 제한하고 대기열을 무한히 두면 메모리와 latency가 계속 증가할 수 있기 때문에 bounded queue, timeout, rejection policy와 같이 설계해야 합니다. 실무에서는 중요한 결제 기능과 상대적으로 중요도가 낮은 이미지 처리 기능이 같은 자원을 모두 공유하지 않도록 분리해서 전체 시스템의 failure domain을 작게 유지하는 것이 핵심입니다.

---

## 18. 기억할 핵심

```text
1. Concurrency Limit = 동시에 실행되는 작업 수의 상한
2. Bulkhead = 기능별 자원 풀을 분리
3. Semaphore만으로는 무한 대기 문제를 해결하지 못한다.
4. Bounded Queue + Timeout + Rejection과 같이 생각한다.
5. 한 기능의 폭주가 전체 시스템 자원을 먹게 두지 않는다.
6. 목표는 failure domain을 작게 만드는 것이다.
```

### 최종 한 문장

> **Bulkhead는 "한 곳이 망가져도 전체가 같이 죽지 않게 만드는 것"이고, Concurrency Limit은 그 격벽 안에서 사용할 수 있는 동시 작업 수에 상한을 두는 것이다.**
