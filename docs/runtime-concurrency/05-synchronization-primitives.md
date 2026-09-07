# 05. Synchronization Primitives

## 한 줄 정의

**Synchronization Primitive은 여러 Thread/Task가 같은 자원에 접근할 때 실행 순서와 동시 접근 범위를 제어하는 도구다.**

대표적으로 `lock`, `Monitor`, `Mutex`, `SemaphoreSlim`, `Interlocked` 등이 있다.

---

## 왜 필요한가

두 Thread가 동시에 같은 값을 수정하면 Race Condition이 생길 수 있다.

```csharp
counter++;
```

이 한 줄은 개념적으로:

```text
read counter
+1
write counter
```

이므로 원자적이라고 볼 수 없다.

## lock

C#의 `lock`은 하나의 critical section에 동시에 한 Thread만 들어가게 한다.

```csharp
lock (_gate)
{
    balance -= amount;
}
```

일반적으로 동일 process 내부 동기화에 사용한다.

### 주의

lock 안에서 오래 걸리는 I/O를 하면 다른 Thread가 모두 기다릴 수 있다.

```csharp
lock (_gate)
{
    CallRemoteApi(); // 위험
}
```

Critical Section은 가능한 작게 유지하는 것이 좋다.

## Monitor

`lock`은 내부적으로 `Monitor.Enter/Exit` 기반 문법 sugar에 가깝다.

직접 Monitor API를 쓰면 더 세밀한 제어가 가능하지만 보통 `lock`이 더 안전하고 간단하다.

## Mutex

Mutex는 상호배제 도구이며 OS-level named mutex 등을 이용하면 process 간 동기화에도 사용할 수 있다.

하지만 process 간 coordination 문제를 모두 Mutex로 해결하는 것이 최선이라는 뜻은 아니다.

분산 시스템에서는 DB constraint, distributed lock, queue 등 다른 도구가 필요할 수 있다.

## Semaphore / SemaphoreSlim

Semaphore는 동시에 들어갈 수 있는 작업 수를 N개로 제한한다.

```csharp
var semaphore = new SemaphoreSlim(10);

await semaphore.WaitAsync();
try
{
    await CallApiAsync();
}
finally
{
    semaphore.Release();
}
```

이 패턴은 외부 API 동시 호출 수나 expensive resource 접근을 제한할 때 유용하다.

`SemaphoreSlim`은 process 내부 async-friendly 동기화에 흔히 쓰인다.

## Interlocked

간단한 atomic 연산에는 `Interlocked`가 유용하다.

```csharp
Interlocked.Increment(ref counter);
```

lock보다 가벼울 수 있지만 모든 복잡한 invariant를 해결하지는 못한다.

예를 들어 여러 필드를 함께 일관되게 바꿔야 한다면 단일 atomic increment만으로 부족하다.

## volatile

`volatile`은 visibility/order와 관련된 의미를 제공하지만 compound operation을 atomic하게 만들지는 않는다.

```csharp
volatile int counter;
counter++; // 여전히 atomic 보장 아님
```

## ReaderWriterLockSlim

Read가 많고 Write가 적은 경우 여러 Reader를 허용하고 Writer는 독점하도록 만들 수 있다.

하지만 실제 성능 이득은 workload에 따라 다르며 복잡도도 증가한다.

## Async 코드에서 lock 주의

`lock` 블록 안에서는 `await`를 사용할 수 없다.

비동기 mutual exclusion이 필요하다면 `SemaphoreSlim(1,1)` 같은 패턴을 사용할 수 있다.

```csharp
await _gate.WaitAsync();
try
{
    await UpdateAsync();
}
finally
{
    _gate.Release();
}
```

다만 이것이 모든 경우에 "async lock"의 완벽한 대체라는 뜻은 아니다. cancellation, fairness, reentrancy를 고려해야 한다.

## Lock Granularity

하나의 전역 lock은 간단하지만 contention이 커질 수 있다.

```text
Global Lock
→ 구현 단순
→ concurrency 낮음

Fine-grained Locks
→ concurrency 증가 가능
→ deadlock/복잡도 증가
```

## Backend에서 중요한 구분

### Process 내부 Race

```text
lock / SemaphoreSlim / Interlocked
```

### 여러 App Instance 간 Race

프로세스 내부 `lock`으로는 해결되지 않는다.

예:

- duplicate order
- inventory decrement
- idempotency

이 경우:

- DB Unique Constraint
- Transaction / Row Lock
- Optimistic Lock
- Distributed coordination

등이 필요하다.

## Deadlock

두 실행 흐름이 서로 가진 lock을 기다리면 deadlock이 생길 수 있다.

```text
Task A: Lock 1 획득 → Lock 2 대기
Task B: Lock 2 획득 → Lock 1 대기
```

예방 핵심:

- consistent lock ordering
- lock 범위 축소
- nested lock 최소화
- I/O를 lock 밖으로 이동

## 60초 면접 답변

> Synchronization primitive은 여러 Thread나 Task가 공유 자원에 동시에 접근할 때 race condition을 제어하는 도구입니다. `lock`은 한 번에 하나만 critical section에 들어가게 하고, `SemaphoreSlim`은 동시 실행 개수를 제한하며 async 코드에서도 사용할 수 있습니다. `Interlocked`는 increment 같은 단순 atomic operation에 적합합니다. 중요한 점은 process 내부 `lock`은 여러 서버 인스턴스 간 race를 해결하지 못하므로, 분산 환경에서는 DB constraint나 transaction 같은 별도 동시성 제어가 필요하다는 것입니다.

## 핵심 오해

- `volatile`은 `counter++`를 atomic하게 만들지 않는다.
- `lock`은 여러 서버 인스턴스 간 동기화를 보장하지 않는다.
- Semaphore는 Mutex와 목적이 다르다.
- lock 범위가 클수록 안전한 것이 아니라 contention 비용도 커진다.
