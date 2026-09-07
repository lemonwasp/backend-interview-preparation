# 10. Memory Visibility와 Memory Barrier

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 설명할 수 있어야 합니다.

- 여러 Thread가 같은 변수를 봐도 왜 항상 같은 값을 즉시 보지 못할 수 있는가?
- CPU Cache, Compiler/CPU Reordering이 왜 동시성 문제를 만든는가?
- Memory Visibility와 Atomicity는 무엇이 다른가?
- Memory Barrier는 무엇을 보장하려는 장치인가?
- C#의 `volatile`, `lock`, `Interlocked`는 어떤 차이가 있는가?

---

## 1. "같은 변수인데 왜 다른 값을 보지?"

멀티코어 CPU에서 각 코어는 메모리를 직접 매번 읽지 않고 Cache를 적극적으로 사용합니다.

```text
Core 1 Cache ─┐
              ├─ Shared Memory
Core 2 Cache ─┘
```

Thread A가 값을 바꿨다고 해서 Thread B가 즉시 그 변경을 관찰한다고 단순하게 가정하면 안 됩니다.

또 Compiler와 CPU는 성능을 위해, 프로그램의 단일 Thread 의미가 깨지지 않는 범위에서 명령 순서를 바꿀 수 있습니다.

즉 동시성에서는 다음 두 문제가 따로 존재합니다.

1. **Visibility**: 다른 Thread의 변경을 언제 볼 수 있는가?
2. **Ordering**: 여러 읽기/쓰기의 순서가 다른 Thread에서도 의도대로 관찰되는가?

---

## 2. Visibility와 Atomicity는 다르다

다음 코드를 봅시다.

```csharp
counter++;
```

이 연산은 보통 개념적으로 다음처럼 나뉩니다.

```text
read counter
add 1
write counter
```

여러 Thread가 동시에 실행하면 Race Condition이 발생할 수 있습니다.
이것은 **Atomicity** 문제입니다.

반면 Thread A가

```csharp
ready = true;
```

라고 쓴 뒤 Thread B가 언제 `true`를 관찰하는지는 **Visibility** 문제입니다.

따라서:

```text
Atomicity ≠ Visibility ≠ Ordering
```

세 개념을 구분해야 합니다.

---

## 3. Reordering은 왜 생길까?

CPU와 Compiler는 성능을 높이기 위해 독립적인 연산의 순서를 바꿀 수 있습니다.

예를 들어 의도는 다음과 같다고 합시다.

```csharp
data = 42;
ready = true;
```

다른 Thread는 `ready == true`를 보면 `data == 42`도 기대할 수 있습니다.

하지만 적절한 동기화가 없다면, 다른 Thread가 관찰하는 순서에 대해 강한 보장을 기대하면 안 됩니다.

이런 이유로 동시성 프로그래밍에서는 **happens-before 관계**를 만들어야 합니다.

---

## 4. Memory Barrier란?

Memory Barrier는 특정 지점 전후의 메모리 연산이 함부로 재배치되거나 보이지 않는 상태로 남지 않도록 순서와 가시성에 제약을 주는 메커니즘입니다.

개념적으로:

```text
write A
write B
--- memory barrier ---
write ready
```

Barrier는 CPU/Compiler 최적화를 완전히 끄는 것이 아니라, 동시성 의미를 지키기 위해 필요한 순서 제약을 만듭니다.

개발자는 보통 CPU 명령 수준의 Barrier를 직접 다루기보다 언어 Runtime의 동기화 도구를 사용합니다.

---

## 5. C#에서는 무엇을 사용할까?

### `lock`

```csharp
lock (gate)
{
    sharedState = newValue;
}
```

Mutual Exclusion뿐 아니라 적절한 Memory Ordering/Visibility도 제공합니다.

### `Interlocked`

```csharp
Interlocked.Increment(ref counter);
```

간단한 원자적 연산에 적합합니다.

### `volatile`

```csharp
private volatile bool _stop;
```

특정 읽기/쓰기에 대해 가시성과 순서 관련 보장을 제공하지만, 여러 연산을 하나의 Atomic 작업으로 만들어주지는 않습니다.

따라서:

```csharp
volatile int counter;
counter++;
```

라고 해도 `counter++` 전체가 원자적이 되는 것은 아닙니다.

---

## 6. `volatile`만으로 해결할 수 없는 이유

여러 값이 함께 일관되게 바뀌어야 한다고 생각해봅시다.

```csharp
balance -= amount;
transactionCount++;
```

이 두 연산 전체가 하나의 Critical Section이어야 한다면 `volatile`만으로는 부족합니다.

이 경우 `lock`, Transaction, 다른 동기화 구조가 필요합니다.

---

## 7. 백엔드에서 어디서 만날까?

- Singleton 내부 Mutable State
- In-memory Cache 갱신
- Background Worker와 HTTP Thread 간 상태 공유
- Connection Pool 상태
- Rate Limiter Counter
- Metrics Counter

특히 "읽기만 하는 것처럼 보여서 안전하겠지"라는 가정은 위험합니다.
공유 상태가 변경된다면 어떤 동기화 규칙으로 Visibility와 Atomicity를 보장하는지 확인해야 합니다.

---

## 8. 자주 하는 오해

### “같은 RAM을 보니까 모든 Thread는 즉시 같은 값을 본다”

아닙니다. CPU Cache, Compiler/CPU 최적화, 메모리 모델 때문에 동기화 규칙이 필요합니다.

### “volatile이면 Thread-safe다”

아닙니다. Visibility와 일부 Ordering 문제를 다루지만 복합 연산의 Atomicity를 자동으로 보장하지 않습니다.

### “lock은 동시에 한 명만 들어가게 하는 기능뿐이다”

아닙니다. 올바른 동기화 경계를 형성하면서 메모리 가시성에도 중요한 역할을 합니다.

---

## 9. 60초 면접 답변

> 멀티스레드 환경에서는 여러 Thread가 같은 변수를 공유해도 CPU Cache와 명령 재배치 때문에 다른 Thread의 변경을 즉시 같은 순서로 관찰한다고 가정할 수 없습니다. 이때 변경이 다른 Thread에 보이는 문제를 Memory Visibility, 연산 순서 문제를 Ordering이라고 합니다. Memory Barrier와 언어의 동기화 도구는 이런 순서와 가시성에 필요한 제약을 만듭니다. C#에서는 보통 `lock`, `Interlocked`, `volatile` 같은 기능을 사용하는데, `volatile`은 복합 연산을 Atomic하게 만드는 기능은 아니므로 Race Condition을 해결하려면 상황에 맞는 동기화 방법을 선택해야 합니다.

---

## 핵심 요약

```text
Atomicity  = 연산이 중간에 끼어들 수 없는가
Visibility = 다른 Thread의 변경을 볼 수 있는가
Ordering   = 연산 순서가 의도대로 관찰되는가
```

다음 주제: Virtual Memory 심화
