# Backpressure and Bounded Queues

## 1. 한 문장으로 설명

**Backpressure는 소비자가 처리할 수 있는 속도보다 생산자가 더 빠를 때, 시스템이 생산 속도를 제어하거나 작업을 제한해서 과부하가 무한히 쌓이지 않게 만드는 메커니즘이다.**

쉽게 말하면:

> 주문이 초당 1,000개 들어오는데 주방은 초당 100개만 처리할 수 있다면, 주문을 무한히 받아서는 안 된다.

백엔드 시스템에서 이 문제를 방치하면 큐, 메모리, 스레드, DB 연결, 외부 API 호출이 계속 쌓이고 결국 전체 시스템이 무너질 수 있다.

---

## 2. 왜 필요한가?

생산 속도와 소비 속도를 각각 다음처럼 생각해보자.

```text
Producer: 1000 req/s
Consumer: 300 req/s
```

매초 처리하지 못한 요청은:

```text
1000 - 300 = 700 requests
```

씩 쌓인다.

10초 뒤에는 약 7,000개, 1분 뒤에는 약 42,000개가 대기하게 된다.

큐가 무한히 커질 수 있다면 처음에는 요청 손실이 없어 보이지만 실제로는 다음 문제가 발생한다.

- 메모리 사용량 증가
- GC 압박 증가
- 응답 지연 폭증
- Thread Pool starvation
- DB Connection Pool 고갈
- timeout 증가
- retry 폭증
- 결국 장애 전파

즉 **무제한 버퍼는 문제를 해결하는 것이 아니라 장애 시간을 뒤로 미루는 것**에 가깝다.

---

## 3. 핵심 구조

```text
Producer
   |
   v
+------------------+
|   Bounded Queue  |
| capacity = 1000  |
+------------------+
   |
   v
Consumer
```

큐가 가득 차면 시스템은 정책을 선택해야 한다.

1. 생산자를 기다리게 한다.
2. 요청을 거절한다.
3. 오래된 작업을 버린다.
4. 새 작업을 버린다.
5. 작업을 샘플링하거나 합친다.
6. 처리량을 늘릴 수 있다면 consumer를 확장한다.

중요한 점은 **'큐가 찼을 때 무엇을 할 것인가'가 시스템 설계의 일부**라는 것이다.

---

## 4. Bounded Queue vs Unbounded Queue

### Unbounded Queue

```text
Producer ---> Queue........................ ---> Consumer
```

장점:

- 구현이 단순하다.
- 순간적인 burst를 흡수할 수 있다.

단점:

- overload 상황에서 메모리 사용량이 제한 없이 증가할 수 있다.
- 오래된 작업이 계속 남아 latency가 폭증한다.
- 장애를 빠르게 드러내지 않고 내부에 축적한다.

### Bounded Queue

```text
Producer ---> [ fixed capacity queue ] ---> Consumer
                    FULL
                     |
                     v
               wait / reject
```

장점:

- 메모리 상한을 예측할 수 있다.
- overload를 빠르게 감지할 수 있다.
- 시스템 전체를 보호하기 쉽다.

단점:

- capacity와 overflow 정책을 설계해야 한다.
- 일부 요청을 거절하거나 지연시켜야 할 수 있다.

실무에서는 일반적으로 **제한된 자원을 다룰 때 bounded 구조가 더 안전하다.**

---

## 5. C# Channel을 이용한 예제

.NET에서는 `System.Threading.Channels`를 사용해 bounded producer-consumer 구조를 만들 수 있다.

```csharp
using System.Threading.Channels;

var channel = Channel.CreateBounded<int>(new BoundedChannelOptions(100)
{
    FullMode = BoundedChannelFullMode.Wait
});

async Task ProduceAsync()
{
    for (int i = 0; i < 10_000; i++)
    {
        await channel.Writer.WriteAsync(i);
    }

    channel.Writer.Complete();
}

async Task ConsumeAsync()
{
    await foreach (var item in channel.Reader.ReadAllAsync())
    {
        await ProcessAsync(item);
    }
}

async Task ProcessAsync(int item)
{
    await Task.Delay(10);
}

await Task.WhenAll(ProduceAsync(), ConsumeAsync());
```

여기서 핵심은 이것이다.

```csharp
Channel.CreateBounded<int>(100)
```

큐에는 최대 100개의 항목만 저장된다.

그리고:

```csharp
FullMode = BoundedChannelFullMode.Wait
```

큐가 가득 차면 `WriteAsync()`가 기다린다.

즉 consumer가 느려지면 producer도 자연스럽게 느려진다.

이것이 backpressure다.

---

## 6. Overflow 정책

.NET bounded channel에는 대표적으로 다음 전략이 있다.

### Wait

```text
Queue Full
    |
Producer waits
```

데이터 손실을 피해야 할 때 적합하다.

예:

- 결제 처리
- 주문 이벤트
- 중요한 작업 큐

### DropWrite

큐가 가득 차면 새 항목을 버린다.

적합한 예:

- 중요하지 않은 telemetry
- 고빈도 metric
- 최신 데이터가 더 중요한 상황

### DropOldest

오래된 항목을 버리고 새 항목을 넣는다.

적합한 예:

- 실시간 위치 정보
- 최신 센서 값
- 화면 갱신 이벤트

핵심은 **업무 의미에 따라 손실 정책이 달라진다**는 것이다.

---

## 7. HTTP 서버에서는 어떻게 나타나는가?

웹 서버에서도 본질은 같다.

```text
Client
  |
  v
API Server
  |
  v
Thread / Task
  |
  v
DB Connection Pool
  |
  v
Database
```

예를 들어 DB Connection Pool이 100개인데 요청이 10,000개 동시에 들어오면 모든 요청을 무조건 처리하려고 하면 안 된다.

가능한 대응:

- 요청 동시성 제한
- queue capacity 제한
- timeout
- rate limiting
- circuit breaker
- load shedding

이 중 **load shedding**은 처리할 수 없는 요청을 의도적으로 빠르게 거절하여 전체 시스템을 보호하는 전략이다.

```text
Overloaded System

slow success for everyone
        X

fast success for some + fast rejection for others
        O
```

극심한 overload에서는 모든 요청을 느리게 성공시키려는 것보다 일부 요청을 빠르게 실패시키는 편이 전체 시스템의 생존에 유리할 수 있다.

---

## 8. Backpressure와 Rate Limiting의 차이

둘은 비슷하지만 목적이 다르다.

### Rate Limiting

외부 요청이 시스템으로 들어오는 속도를 제한한다.

```text
Client -> Rate Limiter -> Server
```

예:

```text
100 requests / second / user
```

### Backpressure

내부 처리 속도가 느려졌을 때 upstream이 그 속도를 따라가도록 만든다.

```text
Producer -> Queue -> Consumer
              FULL
                |
           slow producer
```

정리:

```text
Rate Limiting = 입구에서 속도를 제한
Backpressure  = downstream 상태에 따라 upstream 속도를 조절
```

둘은 함께 사용할 수 있다.

---

## 9. Backpressure와 Retry Storm

장애 상황에서 매우 중요한 연결이다.

```text
Server becomes slow
      |
Clients timeout
      |
Clients retry
      |
Traffic increases
      |
Server becomes slower
      |
More retries
```

이를 **retry storm**이라고 한다.

그래서 실무 시스템에서는 다음을 함께 설계한다.

- bounded queue
- timeout
- exponential backoff
- jitter
- retry limit
- circuit breaker
- rate limiting

Backpressure는 단독 기술이 아니라 **과부하 제어 전략 전체의 일부**다.

---

## 10. Little's Law로 보는 Queue

대략적인 관계는 다음과 같다.

```text
L = λW
```

- `L`: 시스템 안의 평균 요청 수
- `λ`: 단위 시간당 처리되는 요청 수
- `W`: 평균 체류 시간

예를 들어 초당 500개 요청이 처리되고 평균 체류 시간이 2초라면:

```text
L = 500 * 2
  = 1000
```

평균적으로 약 1,000개의 요청이 시스템 안에 존재할 수 있다.

따라서 latency가 증가하면 동시에 시스템 내부에 머무는 요청 수도 증가한다.

이 때문에 overload 상황에서는 단순히 CPU 사용률만 보는 것이 아니라 다음을 같이 봐야 한다.

- queue length
- request latency
- active requests
- Thread Pool queue
- DB pool wait time
- timeout rate
- rejection rate

---

## 11. 장애 시나리오

### 상황

```text
API: 2,000 req/s
DB maximum throughput: 800 req/s
```

### 나쁜 설계

```text
API
 |
 v
Unbounded Queue
 |
 v
DB
```

시간이 지나면:

```text
Queue ↑
Memory ↑
Latency ↑
Timeout ↑
Retry ↑
```

결국 API와 DB가 함께 무너질 수 있다.

### 개선된 설계

```text
Client
  |
Rate Limit
  |
Bounded Queue
  |
Worker Pool
  |
DB Connection Pool
```

큐가 일정 수준을 넘으면:

```text
429 Too Many Requests
503 Service Unavailable
```

등으로 빠르게 실패시키고 클라이언트는 backoff 후 재시도한다.

목표는 **모든 요청을 받는 것**이 아니라 **시스템을 안정적인 처리 가능 영역 안에 유지하는 것**이다.

---

## 12. 면접에서 자주 나오는 질문

### Q1. Backpressure가 무엇인가요?

> 소비자의 처리 속도가 생산자의 생성 속도보다 느릴 때 작업이 무한히 쌓이지 않도록 upstream의 생산 속도를 늦추거나 요청을 제한하는 메커니즘입니다. bounded queue, concurrency limit, load shedding 등을 통해 구현할 수 있습니다.

### Q2. 왜 unbounded queue가 위험한가요?

> 순간적인 burst는 흡수할 수 있지만 지속적인 overload에서는 메모리와 대기 시간이 제한 없이 증가합니다. 결국 GC 압박, timeout, retry storm, 자원 고갈로 이어질 수 있기 때문에 시스템의 실제 capacity를 숨기는 문제가 있습니다.

### Q3. Queue가 가득 차면 어떻게 해야 하나요?

> 업무 특성에 따라 producer를 기다리게 하거나, 요청을 거절하거나, 오래된 작업이나 새 작업을 버리는 정책을 선택합니다. 중요한 데이터는 wait 또는 durable queue가 적합하고, telemetry나 최신 값만 중요한 데이터는 drop 전략도 사용할 수 있습니다.

### Q4. Backpressure와 Rate Limiting의 차이는 무엇인가요?

> Rate limiting은 주로 시스템 입구에서 요청률을 제한하고, backpressure는 downstream의 실제 처리 상태를 upstream에 전달해 생산 속도를 조절한다는 차이가 있습니다.

### Q5. 시스템이 overload되었을 때 왜 빠른 실패가 중요할 수 있나요?

> 처리 능력을 넘어선 요청을 계속 받아들이면 모든 요청의 latency가 증가하고 timeout과 retry가 더 많은 부하를 만들어 장애가 증폭될 수 있습니다. 일부 요청을 빠르게 거절하면 제한된 자원을 정상 요청 처리에 사용할 수 있습니다.

---

## 13. 60초 답변

> Backpressure는 downstream consumer가 처리할 수 있는 속도보다 producer가 빠를 때, 작업이 무한히 쌓이지 않도록 생산 속도를 제한하는 메커니즘입니다. 대표적으로 bounded queue를 두고 큐가 가득 차면 producer를 기다리게 하거나 요청을 거절할 수 있습니다. 이를 하지 않고 unbounded queue를 사용하면 overload 상황에서 메모리, latency, timeout이 계속 증가하고 retry storm까지 발생해 전체 시스템 장애로 이어질 수 있습니다. 실무에서는 bounded queue, concurrency limit, timeout, rate limiting, circuit breaker, load shedding을 함께 사용해 시스템을 처리 가능한 범위 안에 유지합니다.

---

## 14. 기억할 핵심

```text
1. Producer > Consumer가 오래 지속되면 반드시 backlog가 생긴다.
2. Unbounded queue는 overload를 해결하지 않는다.
3. Bounded queue는 시스템에 명확한 상한을 만든다.
4. Queue가 찼을 때의 정책도 설계의 일부다.
5. Backpressure는 시스템 전체를 보호하기 위한 overload control이다.
6. Timeout, retry, rate limit, circuit breaker와 함께 생각해야 한다.
```

### 최종 한 문장

> **Backpressure는 "더 이상 처리할 수 없는데도 계속 받는 시스템"을 "처리 가능한 만큼만 받는 시스템"으로 바꾸는 안전장치다.**
