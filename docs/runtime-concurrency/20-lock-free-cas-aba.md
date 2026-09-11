# Lock-Free Programming, CAS, and the ABA Problem

## 한 줄 정의

**Lock-free programming은 전체 진행이 특정 lock 소유자의 중단에 막히지 않도록 atomic primitive를 사용해 공유 상태를 갱신하는 방식이며, 핵심 도구 중 하나가 Compare-and-Swap(CAS)다.**

---

## 1. CAS란?

CAS는 대략 다음 의미를 가진다.

```text
if current == expected:
    current = newValue
    success
else:
    fail
```

중요한 점은 **비교와 교체가 하나의 atomic operation**이라는 것이다.

.NET에서는 `Interlocked.CompareExchange`를 사용할 수 있다.

```csharp
var old = Interlocked.CompareExchange(
    ref _state,
    newValue,
    expectedValue);

if (old == expectedValue)
{
    // success
}
```

---

## 2. CAS loop

여러 Thread가 같은 값을 갱신하려면 실패 시 다시 읽고 재시도할 수 있다.

```csharp
while (true)
{
    int current = Volatile.Read(ref _value);
    int next = current + 1;

    if (Interlocked.CompareExchange(
            ref _value,
            next,
            current) == current)
    {
        break;
    }
}
```

경쟁에서 진 Thread는 다시 현재 값을 읽고 계산한다.

---

## 3. Lock-free의 의미

`lock-free`는 단순히 "lock 문법을 안 쓴다"는 뜻이 아니다.

일반적으로는:

> 일부 Thread가 지연되거나 멈추더라도 시스템 전체가 계속 progress할 수 있는 성질

을 뜻한다.

다음 개념과 구분할 필요가 있다.

```text
Blocking
Lock-free
Wait-free
```

### Blocking

lock owner가 멈추면 다른 Thread도 기다릴 수 있다.

### Lock-free

개별 Thread는 계속 실패할 수 있지만 전체적으로 누군가는 progress한다.

### Wait-free

각 Thread가 제한된 단계 안에 작업을 완료하도록 보장한다.

Wait-free가 더 강한 조건이다.

---

## 4. Lock-free가 무조건 빠른가?

아니다.

경쟁이 심하면 CAS loop가 계속 실패할 수 있다.

```text
read
CAS fail
retry
CAS fail
retry
...
```

이 경우:

- CPU 사용량 증가
- cache coherence traffic 증가
- starvation 가능성

이 생길 수 있다.

간단한 critical section에서는 lock이 더 읽기 쉽고 충분히 빠를 수 있다.

---

## 5. ABA Problem

CAS의 대표적인 함정이다.

Thread A가 값을 읽는다.

```text
A 읽음: value = A
```

그 사이 Thread B가:

```text
A -> B -> A
```

로 값을 바꾼다.

Thread A가 다시 보면 값은 여전히 A다.

그래서:

```text
current == expected
```

가 성립하지만 **중간에 상태가 바뀌었다가 돌아왔다는 사실**을 놓친다.

이것이 ABA problem이다.

---

## 6. 왜 위험한가

특히 pointer/reference 기반 lock-free stack 같은 구조에서 문제가 된다.

개념적으로:

```text
Top -> Node A -> Node B
```

Thread 1이 A를 읽은 뒤 멈춘다.

Thread 2가:

```text
A pop
B pop
A reuse/push
```

를 수행하면 Top이 다시 A처럼 보일 수 있다.

Thread 1은 "아무것도 안 변했다"고 잘못 판단할 수 있다.

---

## 7. 해결 아이디어: Version Tag

값만 비교하지 않고 버전도 함께 비교한다.

```text
(value=A, version=10)

A -> B -> A

(value=A, version=12)
```

값은 같아도 version이 다르므로 상태 변화가 있었음을 알 수 있다.

이를 tagged pointer / versioned reference 같은 방식으로 구현할 수 있다.

---

## 8. Memory Reclamation 문제

Lock-free 구조에서 더 어려운 문제는 삭제된 node를 언제 안전하게 해제하느냐이다.

다른 Thread가 아직 그 node를 참조하고 있을 수 있다.

대표 아이디어:

- hazard pointer
- epoch-based reclamation
- RCU 계열 접근

GC가 있는 .NET에서는 native pointer 기반 구조보다 메모리 해제 문제가 일부 완화되지만, object lifetime과 상태 재사용 문제를 완전히 무시할 수 있다는 뜻은 아니다.

---

## 9. Spin과 Backoff

CAS 실패가 계속되면 즉시 무한 재시도하는 것보다 backoff를 둘 수 있다.

```text
CAS fail
 -> short spin
 -> yield
 -> retry
```

경쟁이 낮을 때는 빠르게 재시도하고 경쟁이 높을 때는 CPU를 과도하게 태우지 않도록 조정할 수 있다.

---

## 10. 언제 쓰는가

적합한 경우:

- 매우 작은 공유 상태
- 고빈도 counter/state machine
- concurrent queue/stack 내부 구현
- latency-sensitive primitive

부적합할 수 있는 경우:

- 복잡한 business invariant
- 여러 object를 함께 변경
- 유지보수성이 중요한 일반 application code

---

## 11. 면접 답변

> Lock-free 알고리즘은 lock owner의 정지 때문에 전체 progress가 막히지 않도록 CAS 같은 atomic primitive를 사용합니다. `CompareExchange`로 현재 값이 예상 값일 때만 상태를 교체하고 실패하면 다시 읽어 재시도할 수 있습니다. 하지만 경쟁이 높으면 CAS retry 비용이 커질 수 있고, 값이 A에서 B로 바뀌었다가 다시 A로 돌아오면 변화가 없었다고 오인하는 ABA 문제가 있습니다. 이를 version tag 같은 방식으로 완화할 수 있습니다.

## 기억할 핵심

```text
1. CAS = compare + swap을 atomic하게 수행한다.
2. Lock-free != 항상 빠름.
3. Lock-free는 전체 progress 보장에 관한 개념이다.
4. ABA는 A -> B -> A 변화를 놓치는 문제다.
5. 복잡한 비즈니스 로직에 무리하게 lock-free를 쓰지 않는다.
```
