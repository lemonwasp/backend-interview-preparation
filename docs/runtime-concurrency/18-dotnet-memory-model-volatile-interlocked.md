# .NET Memory Model, Volatile, and Interlocked

## 한 줄 정의

**멀티스레드 프로그램에서는 한 Thread가 쓴 값이 다른 Thread에서 언제, 어떤 순서로 보이는지가 중요하며, .NET에서는 `volatile`, `Volatile`, `Interlocked`, lock 등의 도구로 가시성과 원자성을 제어한다.**

---

## 1. 왜 평범한 대입만으로 부족한가

다음 코드를 생각해보자.

```csharp
bool ready = false;
int data = 0;

// Thread A
data = 42;
ready = true;

// Thread B
if (ready)
{
    Console.WriteLine(data);
}
```

직관적으로는 `ready == true`면 `data == 42`일 것 같지만, 멀티스레드에서는 CPU cache, compiler/JIT optimization, instruction reordering 때문에 단순한 소스 코드 순서만으로는 충분하지 않을 수 있다.

핵심 질문은 두 가지다.

```text
1. write가 다른 Thread에 보이는가?        -> visibility
2. 여러 연산이 하나처럼 실행되는가?      -> atomicity
```

둘은 같은 문제가 아니다.

---

## 2. Atomicity와 Visibility

### Atomicity

연산이 중간 상태 없이 하나의 단위처럼 실행되는 성질이다.

```csharp
counter++;
```

는 보통 내부적으로:

```text
read
add
write
```

이므로 여러 Thread가 동시에 실행하면 update가 사라질 수 있다.

### Visibility

한 Thread가 변경한 값을 다른 Thread가 적절한 시점에 관찰할 수 있는가의 문제다.

즉:

```text
atomic != automatically visible in every ordering scenario
visible != compound operation is atomic
```

---

## 3. volatile

```csharp
private volatile bool _stop;
```

`volatile`은 해당 필드 접근에 메모리 ordering/visibility 의미를 부여한다.

대표적인 용도:

```csharp
while (!_stop)
{
    DoWork();
}
```

다른 Thread가:

```csharp
_stop = true;
```

로 변경했을 때 worker가 그 변화를 적절히 관찰하도록 돕는다.

하지만 `volatile`은 복합 연산을 atomic하게 만들어주지 않는다.

```csharp
volatile int counter;
counter++; // 여전히 race 가능
```

---

## 4. Volatile.Read / Volatile.Write

필드 선언 자체를 `volatile`로 만들지 않고 특정 접근에 명시적으로 의미를 줄 수도 있다.

```csharp
Volatile.Write(ref _ready, true);

if (Volatile.Read(ref _ready))
{
    // ...
}
```

라이브러리나 low-level concurrent code에서 의도를 더 명확하게 표현할 때 유용하다.

---

## 5. Interlocked

`Interlocked`는 간단한 공유 상태 변경을 atomic하게 수행한다.

```csharp
Interlocked.Increment(ref _counter);
```

다음과 같은 연산이 있다.

```text
Increment
Decrement
Add
Exchange
CompareExchange
```

예:

```csharp
int oldValue = Interlocked.CompareExchange(
    ref _state,
    newValue,
    expectedValue);
```

의미는 대략:

```text
if state == expected:
    state = newValue
return old state
```

이 비교와 변경이 하나의 atomic operation으로 수행된다.

---

## 6. Compare-and-Swap와 상태 전이

Concurrent code에서는 상태 전이를 다음처럼 만들 수 있다.

```text
Idle -> Running -> Completed
```

두 Thread가 동시에 `Idle -> Running`을 시도해도 CAS를 사용하면 한 Thread만 성공하게 만들 수 있다.

```csharp
if (Interlocked.CompareExchange(
        ref _state,
        Running,
        Idle) == Idle)
{
    StartWork();
}
```

---

## 7. Memory Barrier 직관

CPU와 compiler는 성능을 위해 독립적인 연산의 순서를 바꿀 수 있다.

Memory barrier는 특정 경계를 기준으로 memory operation의 ordering을 제한한다.

개념적으로:

```text
write A
write B
--- barrier ---
read C
```

고수준에서는 직접 barrier를 남발하기보다 `lock`, `Interlocked`, `Volatile`, concurrent collection 같은 검증된 primitive를 사용하는 편이 안전하다.

---

## 8. lock과 비교

```csharp
lock (_gate)
{
    _balance -= amount;
    _history.Add(item);
}
```

`lock`은 여러 연산으로 이루어진 critical section 전체를 보호한다.

반면 `Interlocked`는 하나의 간단한 공유 변수 연산에 특히 적합하다.

```text
단순 counter/state 변경 -> Interlocked 후보
여러 invariant를 함께 보호 -> lock 후보
```

무조건 lock-free가 더 좋은 것은 아니다.

---

## 9. double-checked locking에서의 핵심

Lazy initialization 같은 코드는 memory ordering을 잘못 이해하면 위험하다.

일반 애플리케이션에서는 직접 복잡한 double-checked locking을 구현하기보다:

```csharp
Lazy<T>
```

같은 검증된 abstraction을 사용하는 편이 낫다.

---

## 10. 면접 함정

### Q. volatile int에 `++`하면 thread-safe인가?

아니다.

`++`는 read-modify-write 복합 연산이므로 `Interlocked.Increment` 같은 atomic primitive가 필요하다.

### Q. Interlocked가 있으면 lock은 필요 없나?

아니다.

여러 변수 사이의 invariant를 함께 보호해야 한다면 하나의 atomic integer 변경만으로 충분하지 않을 수 있다.

---

## 11. 60초 답변

> 멀티스레드에서는 공유 메모리의 문제를 atomicity와 visibility로 나눠 생각해야 합니다. `volatile`과 `Volatile.Read/Write`는 주로 값의 가시성과 ordering을 제어하지만 `counter++` 같은 복합 연산을 atomic하게 만들지는 않습니다. 단순 카운터나 상태 전이는 `Interlocked`를 사용할 수 있고, 여러 상태를 하나의 invariant로 보호해야 한다면 `lock`이 더 적절할 수 있습니다. low-level memory ordering을 직접 다루기보다 검증된 synchronization primitive를 사용하는 것이 일반적으로 안전합니다.

## 기억할 핵심

```text
1. Atomicity와 Visibility는 다르다.
2. volatile은 ++를 atomic하게 만들지 않는다.
3. Interlocked는 atomic read-modify-write를 제공한다.
4. CompareExchange는 lock-free 알고리즘의 핵심 primitive다.
5. 복잡한 invariant는 lock이 더 단순하고 안전할 수 있다.
```
