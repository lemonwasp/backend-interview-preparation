# 09. Race Condition, Lock, Deadlock

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 자신의 말로 설명할 수 있어야 합니다.

- Race Condition은 왜 발생하는가?
- Critical Section은 무엇인가?
- Lock/Mutex가 어떤 문제를 해결하는가?
- Atomic Operation은 Lock과 어떻게 다른가?
- Deadlock의 4가지 필요 조건은 무엇인가?
- Deadlock을 어떻게 예방·회피·탐지할 수 있는가?
- 백엔드 코드에서 Lock을 과도하게 사용하면 왜 성능이 나빠지는가?

---

## 1. `counter++`는 정말 한 번에 실행될까?

코드에서는 한 줄입니다.

```csharp
counter++;
```

하지만 개념적으로는 다음과 같은 여러 단계로 나뉠 수 있습니다.

```text
1. counter 읽기
2. +1 계산
3. counter에 다시 쓰기
```

초기값이 0일 때 두 Thread가 동시에 실행하면:

```text
Thread A: read 0
Thread B: read 0
Thread A: write 1
Thread B: write 1
```

두 번 증가했지만 최종값은 1이 됩니다.

이처럼 **실행 순서에 따라 결과가 달라지는 문제**가 Race Condition입니다.

---

## 2. Race Condition의 핵심 조건

Race Condition은 보통 다음 조합에서 나타납니다.

```text
여러 실행 흐름
+ shared mutable state
+ 적절한 synchronization 부재
```

즉 여러 Thread 자체가 문제라기보다 **같은 변경 가능한 상태를 동시에 다루는 것**이
문제의 핵심입니다.

---

## 3. Critical Section

여러 Thread가 동시에 실행하면 안 되는 코드 구간을 Critical Section이라고 합니다.

예:

```text
잔액 읽기
잔액 검증
잔액 감소
새 잔액 저장
```

이 전체가 하나의 논리적 작업이라면 중간에 다른 Thread가 끼어들지 못하도록 보호해야
할 수 있습니다.

---

## 4. Lock / Mutex

Lock은 한 번에 하나의 실행 흐름만 Critical Section에 들어가도록 제한하는 대표적인
동기화 방법입니다.

```text
Thread A ─> lock 획득 ─> critical section ─> unlock
Thread B ─>      기다림       ───────────────> lock 획득
```

C#에서는 예를 들어:

```csharp
private readonly object _gate = new();

lock (_gate)
{
    counter++;
}
```

이렇게 하면 같은 `_gate`를 사용하는 Thread들은 해당 구간을 동시에 실행하지 않습니다.

---

## 5. Lock이 해결하는 것과 만들 수 있는 것

Lock은 Race Condition을 줄이는 데 유용하지만 비용이 있습니다.

- Lock 획득/해제 비용
- 대기 시간
- Thread Blocking
- Context Switching 가능성
- Contention
- Deadlock 위험

따라서 모든 코드를 하나의 큰 Lock으로 감싸면 안전해 보일 수 있지만 동시성이 크게
떨어질 수 있습니다.

```text
큰 Lock
↓
한 번에 한 Thread만 실행
↓
Concurrency 이점 감소
```

---

## 6. Mutex, Monitor, Semaphore의 직관적 차이

정확한 구현은 플랫폼마다 다르지만 개념적으로 다음처럼 구분할 수 있습니다.

### Mutex / Lock

한 번에 한 실행 흐름만 통과시키는 상호 배제 목적입니다.

```text
capacity = 1
```

### Semaphore

동시에 통과 가능한 개수를 N개로 제한합니다.

```text
capacity = N
```

예를 들어 외부 API 호출을 동시에 10개까지만 허용하고 싶다면 Semaphore가 유용할
수 있습니다.

C#의 `lock`은 Monitor 기반 상호 배제 문법입니다.

---

## 7. Atomic Operation

Atomic Operation은 중간 상태가 다른 실행 흐름에 노출되지 않는 하나의 불가분 동작처럼
처리됩니다.

예를 들어 단순 카운터 증가에는 C#의 `Interlocked.Increment` 같은 원자적 연산을
사용할 수 있습니다.

```csharp
Interlocked.Increment(ref counter);
```

장점:

- 단순한 연산에서는 Lock보다 가벼울 수 있다.

하지만 복잡한 여러 단계 비즈니스 로직 전체를 Atomic Operation 하나로 만들 수 있는 것은
아닙니다.

예:

```text
잔액 조회 → 권한 검사 → 한도 확인 → 잔액 차감 → 로그 기록
```

이런 작업은 더 넓은 동기화나 DB Transaction 같은 별도 일관성 메커니즘이 필요할 수
있습니다.

---

## 8. Deadlock

Deadlock은 여러 실행 흐름이 서로가 가진 자원을 기다리면서 영원히 진행하지 못하는
상태입니다.

예를 들어:

```text
Thread A: Lock 1 획득 → Lock 2 기다림
Thread B: Lock 2 획득 → Lock 1 기다림
```

둘 다 상대가 Lock을 놓아주기만 기다립니다.

```text
A holds L1, waits L2
B holds L2, waits L1
```

결과: 아무도 진행하지 못합니다.

---

## 9. Deadlock의 4가지 필요 조건

고전적으로 Deadlock이 성립하려면 다음 조건들이 동시에 존재해야 합니다.

### 1) Mutual Exclusion

자원을 동시에 여러 실행 흐름이 사용할 수 없습니다.

### 2) Hold and Wait

이미 하나의 자원을 가진 상태에서 다른 자원을 기다립니다.

### 3) No Preemption

다른 Thread가 가진 자원을 강제로 빼앗을 수 없습니다.

### 4) Circular Wait

자원 대기 관계가 원형을 이룹니다.

```text
A waits B
B waits C
C waits A
```

이 네 조건 중 하나라도 구조적으로 깨면 Deadlock을 예방할 수 있습니다.

---

## 10. 가장 실용적인 Deadlock 예방: Lock 순서 통일

예를 들어 모든 코드에서 항상:

```text
Lock A → Lock B
```

순서로만 획득하도록 규칙을 정하면 다음과 같은 반대 순서가 나오지 않게 할 수 있습니다.

```text
Thread 1: A → B
Thread 2: B → A   // 위험
```

즉 **Global Lock Ordering**은 Circular Wait를 깨는 대표적인 실무 전략입니다.

---

## 11. Timeout과 Try-Lock

Lock을 영원히 기다리지 않고 일정 시간 뒤 포기하도록 만들 수도 있습니다.

개념적으로:

```text
try lock
if timeout:
    rollback / retry / fail
```

이 방식은 시스템이 무한 대기에 빠지는 것을 줄일 수 있지만 재시도 정책, 일관성, 오류
처리를 함께 설계해야 합니다.

---

## 12. Lock Contention

여러 Thread가 같은 Lock을 자주 경쟁하면 Lock Contention이 발생합니다.

```text
Thread A ─┐
Thread B ─┼─> same lock
Thread C ─┤
Thread D ─┘
```

결과:

- 대기 시간 증가
- CPU 활용률 저하
- Context Switch 증가 가능
- Tail Latency 악화

그래서 Critical Section은 가능한 한 작게 유지하는 것이 일반적으로 유리합니다.

하지만 Lock 범위를 너무 잘게 나누면 Lock 수가 늘어나고 Deadlock 위험과 코드 복잡도가
커질 수 있으므로 균형이 필요합니다.

---

## 13. Shared State 자체를 줄이는 전략

가장 좋은 동시성 제어는 경우에 따라 Lock을 더 정교하게 만드는 것이 아니라 **공유 상태를
줄이는 것**일 수 있습니다.

예:

- Immutable Object
- Message Passing
- Actor Model
- Thread-local State
- 요청 단위 독립 데이터
- Partitioning / Sharding

공유하지 않으면 동기화할 필요도 줄어듭니다.

---

## 14. 백엔드 실무 예시 1 — 재고 감소

잘못된 구현:

```text
재고 = 1

Request A: 재고 조회 → 1
Request B: 재고 조회 → 1
Request A: 구매 성공 → 0 저장
Request B: 구매 성공 → 0 저장
```

재고 하나인데 두 주문이 성공할 수 있습니다.

이 문제는 단일 Process의 Lock만으로 충분하지 않을 수도 있습니다.
여러 서버 Instance가 같은 DB를 사용한다면 DB Transaction, Row Lock, Optimistic
Concurrency Control 같은 분산/DB 수준의 동시성 제어가 필요할 수 있습니다.

즉:

> In-process Lock은 Process 내부만 보호한다.

---

## 15. 백엔드 실무 예시 2 — Cache 갱신

여러 요청이 Cache Miss를 동시에 만나 같은 비싼 DB Query를 실행할 수 있습니다.

```text
100 requests
↓ all cache miss
100 DB queries
```

이를 Cache Stampede라고 부를 수 있습니다.

Lock이나 Single-flight 패턴을 사용해 한 요청만 갱신하고 나머지는 기다리거나 기존 값을
사용하게 만들 수 있습니다.

하지만 Lock을 잘못 설계하면 반대로 요청 전체가 하나의 Lock에 몰려 병목이 생길 수
있습니다.

---

## 16. TIFF-to-PDF 사례와 연결

페이지별 변환을 병렬화할 때 하나의 PDF Document 객체를 여러 Thread가 동시에 수정하면
라이브러리가 Thread-safe하지 않은 경우 문제가 생길 수 있습니다.

안전한 구조의 예:

```text
Parallel:
Page 1 → independent converted buffer
Page 2 → independent converted buffer
Page 3 → independent converted buffer

Then sequential or controlled merge:
buffer1 → PDF
buffer2 → PDF
buffer3 → PDF
```

즉 CPU-heavy 변환 구간은 병렬화하고 Shared State인 최종 PDF 객체 수정은 제어하는 구조를
고려할 수 있습니다.

---

## 17. Deadlock과 Database

Deadlock은 Thread Lock뿐 아니라 Database Transaction에서도 발생할 수 있습니다.

```text
Transaction A: Row 1 lock → Row 2 wait
Transaction B: Row 2 lock → Row 1 wait
```

DBMS는 이런 Deadlock을 탐지해 한 Transaction을 Victim으로 Rollback시키는 전략을 사용할
수 있습니다.

애플리케이션은 Deadlock 오류를 적절히 Retry하도록 설계해야 할 수도 있습니다.

---

## 18. 자주 하는 오해

### “Lock을 쓰면 Thread-safe하다”

올바른 공유 상태와 동일한 Lock을 일관되게 보호해야 합니다. Lock 하나를 썼다는 사실만으로
전체 코드가 Thread-safe해지는 것은 아닙니다.

### “Atomic이면 모든 동시성 문제가 해결된다”

Atomic Operation은 단순 연산에는 유용하지만 복합 상태 일관성까지 자동으로 보장하지
않습니다.

### “Deadlock은 Lock을 두 개 이상 쓸 때만 발생한다”

대표적인 형태는 복수 자원 순환 대기지만 실제 시스템에서는 Thread, DB, 외부 Resource 등
여러 형태의 대기 의존성으로 발생할 수 있습니다.

### “큰 Lock 하나면 가장 안전하니 항상 좋다”

정확성은 단순해질 수 있지만 병렬성과 처리량이 크게 떨어질 수 있습니다.

---

## 19. 60초 면접 답변

> Race Condition은 여러 실행 흐름이 공유된 변경 가능한 상태에 동시에 접근하고
> 실행 순서에 따라 결과가 달라지는 문제입니다. Critical Section을 Lock이나 Mutex로
> 보호해 한 번에 하나의 Thread만 수정하게 만들 수 있지만, Lock에는 대기와 Context
> Switch, Contention 같은 비용이 있습니다. 여러 Lock을 서로 다른 순서로 획득하면
> Deadlock이 발생할 수 있고, Mutual Exclusion, Hold and Wait, No Preemption,
> Circular Wait 네 조건이 동시에 성립할 때 Deadlock이 가능해집니다. 실무에서는 Lock
> 순서를 통일하고 Critical Section을 작게 유지하며, 가능하면 Shared State 자체를 줄이는
> 것이 중요합니다.

---

## 핵심 요약

```text
Race Condition = 실행 순서에 따라 결과가 달라짐
Critical Section = 동시에 실행되면 안 되는 구간
Lock = mutual exclusion 제공
Atomic = 단순 불가분 연산
Deadlock = 서로 자원을 기다리며 영원히 진행 불가
Deadlock 예방 = lock ordering, timeout, shared state 감소 등
```

다음 주제 후보:

- Memory Synchronization / Visibility
- Volatile / Memory Barrier
- Semaphore / Reader-Writer Lock
- Virtual Memory 심화
- Page Replacement / Page Cache
