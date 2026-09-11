# Rate Limiting Algorithms

## 1. 한 문장으로 설명

**Rate Limiting은 일정 시간 동안 허용되는 요청 수를 제한해서 시스템 과부하, 악성 트래픽, 특정 사용자의 자원 독점을 막는 기법이다.**

대표 알고리즘은 다음 네 가지다.

- Fixed Window
- Sliding Window
- Token Bucket
- Leaky Bucket

---

## 2. 왜 필요한가?

예를 들어 API 서버가 초당 1,000개의 요청까지만 안정적으로 처리할 수 있다고 하자.

```text
정상 부하: 800 req/s
폭주 부하: 10,000 req/s
```

제한이 없다면:

```text
request 증가
  ↓
queue 증가
  ↓
latency 증가
  ↓
timeout 증가
  ↓
retry 증가
  ↓
장애 확대
```

Rate Limiting은 이때 시스템 입구에서 처리량을 제한한다.

```text
Client
  |
  v
Rate Limiter
  |
  +--> 허용
  |
  +--> 거절(429)
```

---

## 3. Fixed Window Counter

가장 단순한 방식이다.

예:

```text
1분에 100 요청 허용
```

```text
12:00:00 ~ 12:00:59 -> 최대 100
12:01:00 ~ 12:01:59 -> 다시 최대 100
```

구현 개념:

```text
key = userId + currentMinute
count++

if count > limit:
    reject
```

### 장점

- 구현이 매우 단순하다.
- Redis counter와 잘 맞는다.
- 메모리 사용량이 적다.

### 단점

윈도우 경계에서 burst가 발생할 수 있다.

```text
12:00:59 -> 100 requests
12:01:00 -> 100 requests
```

실제로는 1초 사이에 200개의 요청이 들어올 수 있다.

---

## 4. Sliding Window Log

각 요청의 timestamp를 저장하고 최근 N초 범위 안의 요청 수를 센다.

예:

```text
최근 60초 동안 100개 이하
```

```text
timestamps = [t1, t2, t3, ...]
```

새 요청이 오면:

1. 60초보다 오래된 timestamp 제거
2. 남은 요청 수 확인
3. limit 미만이면 현재 timestamp 추가

### 장점

- 매우 정확하다.
- 경계 burst 문제가 적다.

### 단점

- 요청마다 timestamp를 저장해야 한다.
- 고트래픽 환경에서는 메모리 비용이 크다.

---

## 5. Sliding Window Counter

Fixed Window와 Sliding Window Log의 절충 방식이다.

이전 윈도우와 현재 윈도우의 요청 수를 가중 평균한다.

예:

```text
현재 시점이 윈도우의 25% 지점

estimated = previous * 0.75 + current
```

장점:

- Fixed Window보다 부드럽다.
- Sliding Log보다 메모리 효율이 좋다.

단점:

- 완전히 정확하지는 않다.

---

## 6. Token Bucket

실무에서 매우 자주 쓰이는 방식이다.

버킷 안에 token이 일정 속도로 채워진다고 생각하면 된다.

```text
        token refill
             ↓
      +-------------+
      |  o o o o o  |
      | Token Bucket|
      +-------------+
             |
      request consumes token
```

예:

```text
capacity = 10 tokens
refill = 1 token / second
```

요청 하나당 token 1개를 사용한다.

```text
if token >= 1:
    token--
    allow
else:
    reject
```

### 특징

버킷에 token이 쌓여 있다면 순간적인 burst를 허용할 수 있다.

예:

```text
capacity = 10
refill = 1/s
```

오랫동안 요청이 없었다면 token이 10개까지 쌓인다.

그러면 순간적으로 10개의 요청을 바로 처리할 수 있다.

### 장점

- 평균 요청률을 제한하면서 burst를 허용한다.
- API gateway에서 자주 사용된다.
- 사용자별 quota 구현에 적합하다.

---

## 7. Leaky Bucket

물이 새는 양동이를 떠올리면 된다.

```text
Requests
   ↓
+----------+
|  Queue   |
|  Bucket  |
+----------+
     |
     | 일정 속도
     v
 Processing
```

요청은 버킷에 들어오고 일정한 속도로만 빠져나간다.

예:

```text
input: burst 가능
output: 10 req/s 고정
```

버킷이 꽉 차면 새 요청을 버린다.

### 특징

Token Bucket과 달리 출력 속도를 일정하게 만든다.

```text
Token Bucket
= burst 허용

Leaky Bucket
= 일정한 처리 속도 유지
```

---

## 8. 네 알고리즘 비교

| Algorithm | Burst 허용 | 정확도 | 메모리 | 특징 |
|---|---:|---:|---:|---|
| Fixed Window | 큼 | 낮음 | 낮음 | 가장 단순 |
| Sliding Log | 낮음 | 매우 높음 | 높음 | timestamp 저장 |
| Sliding Counter | 중간 | 높음 | 낮음 | 실용적 절충 |
| Token Bucket | 가능 | 높음 | 낮음 | burst + 평균 rate 제한 |
| Leaky Bucket | 제한 | 높음 | queue 필요 | 일정한 출력률 |

---

## 9. Token Bucket 간단 구현

```csharp
public class TokenBucket
{
    private readonly int _capacity;
    private readonly double _refillPerSecond;

    private double _tokens;
    private DateTime _lastRefill;
    private readonly object _lock = new();

    public TokenBucket(int capacity, double refillPerSecond)
    {
        _capacity = capacity;
        _refillPerSecond = refillPerSecond;
        _tokens = capacity;
        _lastRefill = DateTime.UtcNow;
    }

    public bool TryAcquire()
    {
        lock (_lock)
        {
            Refill();

            if (_tokens < 1)
                return false;

            _tokens -= 1;
            return true;
        }
    }

    private void Refill()
    {
        var now = DateTime.UtcNow;
        var elapsed = (now - _lastRefill).TotalSeconds;

        _tokens = Math.Min(
            _capacity,
            _tokens + elapsed * _refillPerSecond
        );

        _lastRefill = now;
    }
}
```

핵심은:

```text
현재 token 수
+ 경과 시간 × refill rate
```

를 계산하고 capacity를 넘지 않게 유지하는 것이다.

---

## 10. 분산 환경에서는 왜 어려운가?

서버가 한 대가 아니라 여러 대라면 문제가 생긴다.

```text
Client
  |
Load Balancer
  |
  +--> Server A
  +--> Server B
  +--> Server C
```

각 서버가 독립적으로 100 req/min 제한을 두면 실제 전체 제한은 최대 300 req/min이 될 수 있다.

따라서 중앙 상태 저장소가 필요할 수 있다.

```text
Server A ─┐
Server B ─┼--> Redis
Server C ─┘
```

Redis를 사용할 때는 다음이 중요하다.

- atomic increment
- expiration
- race condition 방지
- Lua script 또는 atomic operation
- network latency
- Redis 장애 시 정책

---

## 11. 무엇을 key로 제한할 것인가?

Rate Limit은 알고리즘뿐 아니라 key 설계도 중요하다.

예:

```text
IP
User ID
API Key
Tenant ID
Endpoint
Region
```

실무에서는 조합하기도 한다.

```text
user:123:/payments
```

즉 사용자 123의 `/payments` 호출만 별도로 제한할 수 있다.

---

## 12. 429 Too Many Requests

요청이 제한되면 일반적으로:

```text
HTTP 429 Too Many Requests
```

를 반환한다.

클라이언트가 언제 다시 시도해야 하는지 알려주기 위해:

```text
Retry-After
```

헤더를 사용할 수 있다.

중요한 점은 클라이언트가 바로 무한 재시도를 하지 않도록 하는 것이다.

```text
429
 ↓
backoff
 ↓
retry
```

---

## 13. Rate Limiting vs Concurrency Limiting

둘은 비슷해 보이지만 다르다.

### Rate Limit

```text
초당 몇 개까지?
```

예:

```text
100 req/s
```

### Concurrency Limit

```text
동시에 몇 개까지?
```

예:

```text
max 50 concurrent requests
```

느린 요청이 많다면 초당 요청 수가 낮더라도 concurrency가 크게 증가할 수 있다.

그래서 둘을 함께 쓰는 경우가 많다.

```text
Rate Limit
   ↓
Concurrency Limit
   ↓
Bounded Queue
   ↓
Backend
```

---

## 14. Rate Limiting vs Backpressure

```text
Rate Limiting
= 입구에서 요청률 제한

Backpressure
= downstream 처리 속도에 맞춰 upstream 속도 조절
```

Rate Limit은 정책 기반이고, Backpressure는 실제 시스템 상태에 반응하는 경우가 많다.

---

## 15. 면접 질문

### Q1. Token Bucket과 Leaky Bucket의 차이는?

> Token Bucket은 일정 속도로 token을 충전하고 token이 남아 있으면 순간적인 burst를 허용합니다. 반면 Leaky Bucket은 queue에서 일정 속도로 요청을 내보내기 때문에 출력률을 평탄하게 유지하는 데 더 적합합니다.

### Q2. Fixed Window의 문제는?

> 윈도우 경계에서 두 구간의 quota를 연속으로 사용할 수 있어 실제 짧은 시간 동안 허용량의 두 배 가까운 burst가 발생할 수 있습니다.

### Q3. 분산 환경에서 Rate Limiter를 어떻게 구현하나요?

> 여러 서버가 동일한 제한 상태를 공유해야 하므로 Redis 같은 중앙 저장소를 사용할 수 있습니다. 이때 counter 증가와 만료 처리는 atomic해야 하고, 저장소 장애 시 fail-open 또는 fail-closed 정책도 결정해야 합니다.

### Q4. Rate Limit과 Concurrency Limit의 차이는?

> Rate Limit은 시간당 요청 수를 제한하고 Concurrency Limit은 동시에 실행 중인 요청 수를 제한합니다. 느린 요청이 많을 때는 요청률이 낮아도 동시 실행 수가 커질 수 있기 때문에 둘은 서로 다른 문제를 해결합니다.

---

## 16. 60초 답변

> Rate Limiting은 일정 시간 동안 허용되는 요청 수를 제한해 시스템 과부하와 자원 독점을 방지하는 기법입니다. 단순한 Fixed Window는 구현이 쉽지만 경계 burst 문제가 있고, Sliding Window는 더 정확하지만 비용이 큽니다. Token Bucket은 일정 속도로 token을 충전하면서 남은 token만큼 burst를 허용하기 때문에 API rate limiter에서 자주 사용되고, Leaky Bucket은 요청을 일정 속도로 배출해 처리율을 평탄하게 만드는 데 적합합니다. 분산 환경에서는 여러 서버가 동일한 제한 상태를 공유해야 하므로 Redis 같은 중앙 저장소와 atomic 연산을 사용하며, Rate Limit은 Concurrency Limit과 Backpressure와 함께 사용해 시스템을 보호합니다.

---

## 17. 기억할 핵심

```text
Fixed Window
= 단순하지만 경계 burst

Sliding Window
= 더 정확하지만 비용 증가

Token Bucket
= 평균 rate 제한 + burst 허용

Leaky Bucket
= 일정한 출력률

Rate Limit
= 시간당 요청 수 제한

Concurrency Limit
= 동시에 실행되는 요청 수 제한
```

### 최종 한 문장

> **좋은 Rate Limiter는 단순히 요청을 막는 장치가 아니라, 시스템 capacity를 예측 가능한 범위 안에 유지하는 admission control이다.**
