# False Sharing and CPU Cache Coherence

## 한 줄 정의

**False Sharing은 서로 다른 Thread가 논리적으로 다른 변수를 수정하더라도, 그 변수들이 같은 CPU cache line에 있으면 불필요한 cache invalidation이 반복되어 성능이 크게 떨어지는 현상이다.**

---

## 1. CPU cache를 먼저 생각하자

CPU는 메인 메모리보다 훨씬 빠른 cache를 사용한다.

대략적인 계층은 다음과 같다.

```text
Register
  ↓
L1 Cache
  ↓
L2 Cache
  ↓
L3 Cache
  ↓
RAM
```

CPU는 보통 변수를 한 바이트씩 가져오는 것이 아니라 **cache line** 단위로 가져온다.

일반적으로 cache line은 64 bytes인 경우가 많다.

---

## 2. 문제 상황

다음 두 counter를 생각해보자.

```csharp
long counterA;
long counterB;
```

Thread A는 `counterA`만 수정하고 Thread B는 `counterB`만 수정한다.

논리적으로는 공유하지 않는 것처럼 보인다.

하지만 두 변수가 같은 cache line에 있다면:

```text
Cache Line
+-------------------------------+
| counterA | counterB | ...     |
+-------------------------------+
```

Thread A가 counterA를 수정할 때 해당 cache line 전체의 ownership 상태가 바뀔 수 있다.

Thread B도 counterB를 수정하려면 같은 line을 다시 가져와야 한다.

```text
Core A write
   ↓
Core B cache invalidated
   ↓
Core B write
   ↓
Core A cache invalidated
   ↓
repeat...
```

이것이 false sharing이다.

---

## 3. 왜 'False' Sharing인가

실제 프로그램 관점에서는:

```text
Thread A -> counterA
Thread B -> counterB
```

서로 다른 데이터를 사용한다.

하지만 하드웨어 관점에서는 같은 cache line을 공유한다.

즉 **논리적 공유는 없는데 물리적 cache 단위 때문에 공유처럼 동작**한다.

---

## 4. Cache Coherence

멀티코어 CPU는 각 코어가 가진 cache의 값이 모순되지 않도록 coherence protocol을 사용한다.

중요한 직관은 다음이다.

```text
한 Core가 특정 cache line을 수정하려면
다른 Core가 가지고 있는 같은 line의 사본을 무효화해야 할 수 있다.
```

따라서 여러 Core가 같은 line을 계속 쓰면 cache line이 Core 사이를 ping-pong하게 된다.

---

## 5. 코드 예시

```csharp
public sealed class Counters
{
    public long A;
    public long B;
}
```

두 Thread가 각각 다음을 반복한다고 하자.

```csharp
for (int i = 0; i < 100_000_000; i++)
{
    counters.A++;
}
```

```csharp
for (int i = 0; i < 100_000_000; i++)
{
    counters.B++;
}
```

변수 배치에 따라 예상보다 훨씬 느려질 수 있다.

---

## 6. Padding 아이디어

한 해결 방법은 hot field 사이에 padding을 두어 서로 다른 cache line으로 분리하는 것이다.

개념적으로:

```text
Line 1: counterA + padding
Line 2: counterB + padding
```

다만 실제 .NET object layout은 runtime과 platform 영향을 받으므로 단순히 padding field 몇 개를 넣었다고 항상 원하는 배치가 보장된다고 생각하면 안 된다.

low-level 성능 최적화에서는 layout을 측정하고 검증해야 한다.

---

## 7. Contention과의 차이

### True Sharing

여러 Thread가 실제로 같은 변수를 수정한다.

```text
Thread A -> counter
Thread B -> counter
```

### False Sharing

서로 다른 변수를 수정하지만 같은 cache line에 존재한다.

```text
Thread A -> counterA
Thread B -> counterB
         같은 cache line
```

둘 다 성능 저하를 만들 수 있지만 원인이 다르다.

---

## 8. 언제 문제가 되는가

False sharing은 특히 다음 상황에서 중요해질 수 있다.

- 고빈도 metric counter
- lock-free queue
- ring buffer
- thread-local statistics를 인접 배열에 저장
- high-throughput trading / telemetry
- 게임 엔진
- low-latency server

일반 CRUD 애플리케이션에서는 먼저 DB, network, allocation, algorithm 병목을 보는 것이 보통 더 중요하다.

---

## 9. 배열에서도 발생할 수 있다

예를 들어 Thread별 counter를 배열에 둔다.

```csharp
long[] counters = new long[8];
```

각 Thread가 자기 index만 수정해도 인접 원소가 같은 cache line에 묶일 수 있다.

```text
Core0 -> counters[0]
Core1 -> counters[1]
Core2 -> counters[2]
```

논리적으로 thread-local이지만 물리적으로는 false sharing이 발생할 수 있다.

---

## 10. 측정이 중요하다

False sharing은 코드만 보고 확정하기 어렵다.

다음과 같이 접근한다.

```text
1. benchmark
2. CPU profiling
3. contention/cache 관련 counter 확인
4. layout 변경
5. 다시 benchmark
```

성능 최적화는 추측보다 측정이 우선이다.

---

## 11. 면접 답변

> False sharing은 여러 Thread가 서로 다른 변수를 수정하지만 그 변수들이 같은 CPU cache line에 위치해서 cache coherence traffic이 반복적으로 발생하는 현상입니다. 한 Core가 line을 수정할 때 다른 Core의 같은 line 사본이 invalidation되면서 cache line이 Core 사이를 오가게 되고 처리량이 떨어질 수 있습니다. 해결은 데이터 배치를 분리하거나 per-thread state를 적절히 구성하는 방식이 있지만, 실제 병목인지 benchmark와 profiling으로 검증해야 합니다.

## 기억할 핵심

```text
1. CPU는 cache line 단위로 데이터를 다룬다.
2. 다른 변수라도 같은 line이면 서로 영향을 줄 수 있다.
3. False sharing은 correctness 문제가 아니라 주로 performance 문제다.
4. True sharing과 구분해야 한다.
5. 최적화 전 반드시 측정한다.
```
