# Circuit Breaker, Timeout, Retry, Backoff, and Jitter

## 1. 한 문장으로 설명

분산 시스템에서 외부 서비스가 느려지거나 실패할 때 무작정 기다리거나 계속 재시도하면 장애가 증폭될 수 있다. 그래서 **Timeout으로 기다리는 시간을 제한하고, Retry는 제한적으로 수행하며, Backoff와 Jitter로 재시도 간격을 분산하고, Circuit Breaker로 실패 중인 대상에 대한 호출 자체를 잠시 차단한다.**

---

## 2. 왜 필요한가?

예를 들어 API 서버가 결제 서버를 호출한다고 하자.

```text
Client
  |
  v
API Server
  |
  v
Payment Service
```

Payment Service가 평소에는 100ms에 응답하지만 장애 때문에 30초씩 걸린다면 API Server의 요청도 함께 묶인다.

```text
slow downstream
      ↓
requests waiting
      ↓
threads/tasks/connections occupied
      ↓
queue grows
      ↓
timeout + retry 증가
      ↓
더 많은 부하
```

즉 작은 장애가 전체 시스템 장애로 커질 수 있다.

---

## 3. Timeout

Timeout은 "얼마나 오래 기다릴 것인가"를 정한다.

```text
request
  |
  +---- response within 2s ---> success
  |
  +---- no response ----------> timeout
```

Timeout이 없으면 느린 downstream 때문에 호출자가 자원을 오랫동안 점유할 수 있다.

### 중요한 점

Timeout은 짧다고 무조건 좋은 것도, 길다고 좋은 것도 아니다.

너무 짧으면 정상 요청도 실패한다.
너무 길면 장애 시 자원이 오래 묶인다.

따라서 실제 latency 분포와 SLA/SLO를 기준으로 정해야 한다.

---

## 4. Retry

일시적인 실패라면 재시도가 성공할 수 있다.

예:

- 순간적인 네트워크 오류
- 일시적인 503
- 짧은 lock contention
- leader failover 직후

하지만 모든 실패를 재시도하면 안 된다.

```text
400 Bad Request -> 재시도 의미 없음
401 Unauthorized -> 인증 문제 해결 전 재시도 의미 없음
500/503 -> 경우에 따라 재시도 가능
Timeout -> idempotency를 고려해 재시도 판단
```

### 가장 중요한 조건: Idempotency

같은 요청을 여러 번 보내도 결과가 안전해야 한다.

예를 들어 결제 요청을 단순 retry하면 중복 결제가 발생할 수 있다.

그래서 결제·주문 같은 요청은 보통 idempotency key를 함께 사용한다.

```text
POST /payments
Idempotency-Key: order-12345
```

---

## 5. 왜 즉시 Retry가 위험한가?

서버가 과부하 상태인데 모든 클라이언트가 즉시 재시도하면:

```text
Server fails
   ↓
1000 clients retry immediately
   ↓
server receives even more traffic
   ↓
server becomes slower
   ↓
more timeouts
   ↓
more retries
```

이것이 Retry Storm이다.

따라서 retry는 반드시 속도를 조절해야 한다.

---

## 6. Exponential Backoff

재시도 간격을 점점 늘린다.

예:

```text
1st retry: 100ms
2nd retry: 200ms
3rd retry: 400ms
4th retry: 800ms
5th retry: 1600ms
```

대략:

```text
delay = baseDelay * 2^attempt
```

장점은 장애 중인 서버가 회복할 시간을 준다는 것이다.

---

## 7. Jitter

Backoff만 사용해도 문제가 남는다.

모든 클라이언트가 같은 시간에 실패했다면 같은 backoff를 계산하고 다시 동시에 요청할 수 있다.

```text
1000 clients fail at 12:00:00
all retry after 1 second
      ↓
12:00:01에 다시 1000 requests
```

이를 피하기 위해 무작위 지연을 추가한다.

```text
retry delay = backoff + random jitter
```

예:

```text
Client A -> 920ms
Client B -> 1130ms
Client C -> 1470ms
Client D -> 1040ms
```

이렇게 재시도 시점을 분산한다.

---

## 8. Circuit Breaker

Circuit Breaker는 실패 중인 서비스에 계속 요청을 보내지 않게 한다.

전기 차단기처럼 생각하면 된다.

### Closed

정상 상태.

```text
request -> downstream -> response
```

### Open

실패가 일정 기준을 넘으면 회로를 연다.

```text
request -> immediately fail
```

downstream에 실제 요청을 보내지 않는다.

### Half-Open

일정 시간이 지나면 소수의 요청만 시험적으로 보낸다.

```text
few test requests
   |
   +-- success -> Closed
   |
   +-- failure -> Open
```

---

## 9. Circuit Breaker 상태 흐름

```text
        failures exceed threshold
Closed --------------------------> Open
  ^                                 |
  |                                 | cooldown
  |                                 v
  +----------- success ---------- Half-Open
                   |
                   +-- failure --> Open
```

핵심은 **실패 중인 dependency에 요청을 계속 보내지 않는 것**이다.

---

## 10. C#에서의 간단한 개념 예시

실무에서는 Polly 같은 resilience library를 많이 사용하지만, 개념적으로는 다음 구조다.

```csharp
async Task<T> ExecuteWithRetryAsync<T>(Func<Task<T>> action)
{
    int maxAttempts = 3;
    int delayMs = 100;

    for (int attempt = 1; attempt <= maxAttempts; attempt++)
    {
        try
        {
            return await action();
        }
        catch when (attempt < maxAttempts)
        {
            int jitter = Random.Shared.Next(0, 100);
            await Task.Delay(delayMs + jitter);
            delayMs *= 2;
        }
    }

    throw new InvalidOperationException("Unreachable");
}
```

핵심은 문법보다 정책이다.

```text
How many retries?
Which errors are retryable?
How long is timeout?
Is operation idempotent?
How much backoff?
How much jitter?
When should circuit open?
```

---

## 11. Timeout + Retry의 함정

예를 들어 timeout이 2초이고 retry를 3번 한다고 하자.

최악의 경우:

```text
2s + 2s + 2s = 6s
```

여기에 backoff까지 들어가면 총 latency는 더 길어진다.

따라서 개별 시도 timeout만 보는 것이 아니라 **전체 request deadline**을 같이 봐야 한다.

```text
Total deadline: 3 seconds

Attempt 1: max 1s
Backoff: 200ms
Attempt 2: remaining budget
```

이런 식으로 latency budget 안에서 정책을 구성해야 한다.

---

## 12. Retry Budget

Retry는 공짜가 아니다.

원래 요청이 1000 req/s인데 모두 한 번씩 retry하면 최대 2000 req/s가 될 수 있다.

그래서 대규모 시스템에서는 retry 자체에 제한을 둔다.

예:

```text
normal traffic: 1000 req/s
retry budget: +10%
maximum retry traffic: 100 req/s
```

이렇게 해야 장애 시 retry가 새로운 장애 원인이 되는 것을 막을 수 있다.

---

## 13. Bulkhead와 함께 쓰기

이전 문서의 Bulkhead와 연결해보자.

```text
Client
  |
Timeout
  |
Retry + Backoff + Jitter
  |
Circuit Breaker
  |
Concurrency Limit
  |
Bounded Queue
  |
Dependency
```

각각 역할이 다르다.

```text
Timeout           = 너무 오래 기다리지 않음
Retry             = 일시적 실패를 다시 시도
Backoff           = 재시도 간격을 증가
Jitter            = 재시도 시점을 분산
Circuit Breaker   = 실패 중인 대상 호출을 잠시 차단
Concurrency Limit = 동시에 실행하는 수 제한
Bounded Queue     = 대기 작업 수 제한
Bulkhead          = 자원 풀을 분리해 장애 격리
```

이 조합이 분산 시스템의 기본적인 resilience 패턴이다.

---

## 14. 실전 장애 시나리오

### 나쁜 설계

```text
API -> Payment
```

Payment가 느려진다.

API는 timeout 없이 기다린다.
요청이 쌓인다.
Thread/connection이 고갈된다.
클라이언트가 다시 요청한다.
전체 API가 느려진다.

### 개선된 설계

```text
API
 |
 +-- Timeout: 1s
 +-- Retry: max 2
 +-- Exponential Backoff
 +-- Jitter
 +-- Circuit Breaker
 +-- Concurrency Limit
 +-- Idempotency Key
 |
Payment
```

Payment 장애가 발생해도 API 전체가 같이 무너질 가능성을 줄일 수 있다.

---

## 15. 면접 질문

### Q1. Circuit Breaker가 왜 필요한가요?

> 실패 중인 downstream에 계속 요청을 보내면 timeout과 자원 점유가 누적되고 장애가 증폭될 수 있습니다. Circuit Breaker는 실패율이 일정 기준을 넘으면 호출을 빠르게 실패시켜 downstream과 호출자 양쪽을 보호합니다.

### Q2. Retry는 언제 사용하면 안 되나요?

> 영구적인 오류나 잘못된 요청에는 의미가 없고, non-idempotent 작업은 중복 실행 위험이 있기 때문에 주의해야 합니다. 또한 과도한 retry는 retry storm을 만들 수 있습니다.

### Q3. Exponential Backoff만 쓰면 충분한가요?

> 아닙니다. 많은 클라이언트가 동시에 실패하면 같은 backoff 시점에 다시 몰릴 수 있어서 jitter를 추가해 재시도 시점을 분산하는 것이 좋습니다.

### Q4. Timeout과 Circuit Breaker의 차이는 무엇인가요?

> Timeout은 개별 호출이 얼마나 오래 기다릴지를 제한하고, Circuit Breaker는 최근 실패 상태를 기반으로 일정 기간 호출 자체를 차단합니다.

### Q5. Retry 횟수를 무조건 늘리면 성공률이 올라가나요?

> 일시적 오류에는 도움이 될 수 있지만 장애 상태에서는 부하를 더 키울 수 있습니다. retry 횟수, backoff, jitter, 전체 deadline, retry budget을 함께 설계해야 합니다.

---

## 16. 60초 답변

> 외부 서비스 호출에서는 timeout, retry, backoff, jitter, circuit breaker를 함께 고려합니다. Timeout은 한 호출이 너무 오래 자원을 점유하지 않게 하고, retry는 일시적 실패를 복구하지만 반드시 retry 가능한 오류와 idempotency를 확인해야 합니다. 재시도는 exponential backoff로 간격을 늘리고 jitter를 추가해 여러 클라이언트가 동시에 재시도하는 retry storm을 막습니다. 실패가 계속되면 circuit breaker를 열어 downstream 호출을 잠시 차단하고 빠르게 실패시킵니다. 이 정책들은 concurrency limit, bounded queue, bulkhead와 함께 사용해 장애 전파를 줄입니다.

---

## 17. 기억할 핵심

```text
1. Timeout 없는 외부 호출은 위험하다.
2. Retry는 모든 오류에 적용하는 것이 아니다.
3. Retry 전에 idempotency를 확인한다.
4. Backoff는 서버의 회복 시간을 준다.
5. Jitter는 동시 재시도 폭주를 분산한다.
6. Circuit Breaker는 실패 중인 dependency를 잠시 차단한다.
7. 개별 timeout보다 전체 deadline을 함께 본다.
8. Retry에도 budget이 필요하다.
9. Resilience는 하나의 패턴이 아니라 여러 보호 장치의 조합이다.
```

### 최종 한 문장

> **좋은 분산 시스템은 실패를 없애는 시스템이 아니라, 실패가 발생했을 때 기다림·재시도·부하·장애 전파를 통제하는 시스템이다.**
