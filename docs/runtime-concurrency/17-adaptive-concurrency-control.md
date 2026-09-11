# Adaptive Concurrency Control

## 한 줄 정의

**Adaptive Concurrency Control은 시스템의 지연시간과 실패율을 관찰하면서 동시에 처리하는 요청 수를 동적으로 조절하는 과부하 제어 방식이다.**

고정된 동시성 제한이 `항상 최대 50개`라면, adaptive 방식은 현재 시스템 상태에 따라 20개, 40개, 80개처럼 허용량을 바꾼다.

---

## 1. 왜 고정 제한만으로 부족한가

서버의 실제 처리 능력은 항상 같지 않다.

- DB가 느려질 수 있다.
- 외부 API latency가 증가할 수 있다.
- GC pause가 발생할 수 있다.
- CPU contention이 커질 수 있다.
- 특정 시간대에 cache hit ratio가 달라질 수 있다.

따라서 평소에는 100개 동시 요청을 감당하던 서버가 장애 직전에는 30개만 안정적으로 처리할 수도 있다.

```text
정상 상태: concurrency 100 -> latency 80ms
DB 저하:   concurrency 100 -> latency 2s
```

이때 계속 100개를 허용하면 대기 요청과 timeout이 쌓인다.

---

## 2. 핵심 아이디어

시스템은 보통 다음 신호를 본다.

- p50 / p95 / p99 latency
- timeout rate
- error rate
- queue length
- in-flight requests
- downstream saturation

그리고 concurrency limit을 조절한다.

```text
latency 안정
   -> 조금 증가

latency 급증 / timeout 증가
   -> 빠르게 감소
```

이 패턴은 TCP congestion control과 비슷한 직관을 가진다.

---

## 3. AIMD 직관

가장 이해하기 쉬운 방식 중 하나는 AIMD다.

```text
AIMD = Additive Increase, Multiplicative Decrease
```

정상 상태에서는 천천히 늘린다.

```text
20 -> 21 -> 22 -> 23
```

문제가 감지되면 크게 줄인다.

```text
80 -> 40
```

이유는 간단하다.

> 여유 capacity는 천천히 탐색하고, overload는 빠르게 탈출한다.

---

## 4. 단순 C# 예시

```csharp
public sealed class AdaptiveLimiter
{
    private int _limit = 20;
    private const int Min = 1;
    private const int Max = 200;

    public int CurrentLimit => Volatile.Read(ref _limit);

    public void RecordSuccess(TimeSpan latency)
    {
        if (latency < TimeSpan.FromMilliseconds(200))
        {
            Interlocked.Exchange(
                ref _limit,
                Math.Min(Max, CurrentLimit + 1));
        }
    }

    public void RecordOverload()
    {
        Interlocked.Exchange(
            ref _limit,
            Math.Max(Min, CurrentLimit / 2));
    }
}
```

실제 구현은 이것보다 훨씬 신중해야 하지만 핵심은 다음이다.

```text
좋은 상태 -> limit 증가
나쁜 상태 -> limit 감소
```

---

## 5. 왜 latency를 봐야 하나

CPU가 50%라고 해서 서버가 건강한 것은 아니다.

예를 들어 DB connection pool이 가득 차면:

```text
CPU = 40%
DB pool wait = 2 sec
API p99 = 3 sec
```

CPU만 보면 여유가 있어 보이지만 실제 요청 경로는 이미 포화 상태다.

따라서 concurrency control은 **요청이 실제로 경험하는 latency**를 중요한 신호로 사용한다.

---

## 6. Queue와의 관계

동시성 제한 없이 긴 queue만 두면:

```text
arrival > service capacity
        ↓
queue grows
        ↓
latency grows
        ↓
timeout grows
```

Adaptive concurrency는 queue가 폭증하기 전에 in-flight 작업 수 자체를 조절하는 역할을 한다.

```text
Admission
   ↓
Adaptive Limit
   ↓
Workers
   ↓
Downstream
```

---

## 7. Little's Law 연결

```text
L = λW
```

- L: 시스템 내부 평균 요청 수
- λ: 처리율
- W: 평균 체류 시간

처리율이 비슷한데 latency가 10배 증가하면 시스템 내부에 머무는 요청 수도 크게 늘어난다.

즉 latency 증가는 단순한 사용자 경험 문제가 아니라 **동시 점유 자원의 증가 신호**이기도 하다.

---

## 8. Concurrency Limit과 Rate Limit의 차이

```text
Rate Limit
= 시간당 몇 개를 받을 것인가

Concurrency Limit
= 지금 동시에 몇 개를 처리할 것인가
```

예:

```text
Rate: 1000 requests/sec
Concurrency: max 50 in-flight
```

둘은 서로 다른 축이다.

---

## 9. Adaptive Limit이 위험한 경우

잘못 설계하면 oscillation이 생길 수 있다.

```text
limit 증가
 -> latency 증가
 -> limit 급감
 -> latency 감소
 -> 다시 급증
```

그래서 실무에서는 보통:

- 이동 평균
- percentile latency
- cooldown
- minimum sample count
- hysteresis

등을 사용해 노이즈에 과민 반응하지 않게 한다.

---

## 10. 면접 답변

> Adaptive concurrency control은 고정된 동시 처리 수를 사용하는 대신 실제 latency, timeout, error 같은 신호를 보고 in-flight request limit을 동적으로 조정하는 방식입니다. 시스템이 안정적이면 limit을 조금씩 늘리고 overload가 감지되면 빠르게 줄여 queue 폭증과 cascading failure를 막습니다. Rate limiting이 시간당 요청 수를 제한한다면 concurrency limiting은 특정 순간 동시에 점유되는 작업 수를 제한한다는 차이가 있습니다.

## 기억할 핵심

```text
1. 시스템 capacity는 고정값이 아니다.
2. latency는 saturation의 중요한 신호다.
3. 정상일 때 천천히 늘리고 overload 때 빠르게 줄인다.
4. queue가 커진 뒤 대응하기보다 in-flight 수를 먼저 제어한다.
5. Rate와 Concurrency는 서로 다른 제한이다.
```
