# 05. Context Switching

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 자신의 말로 설명할 수 있어야 합니다.

- Context Switch란 무엇인가?
- 운영체제는 왜 실행 중인 Thread를 바꾸는가?
- 어떤 상태를 저장하고 복원하는가?
- Context Switch에는 왜 비용이 드는가?
- Mode Switch와 Context Switch는 무엇이 다른가?
- 백엔드에서 Thread 수가 너무 많으면 왜 문제가 되는가?

---

## 1. CPU는 한 번에 모든 일을 할 수 없다

하나의 CPU Core는 한 순간에 하나의 명령 흐름을 실행합니다.

여러 Process와 Thread가 동시에 실행되는 것처럼 보이는 이유는 운영체제가 매우 빠르게 실행 대상을 바꾸기 때문입니다.

```text
시간 →

Thread A: [실행]      [실행]
Thread B:      [실행]      [실행]
Thread C:           [실행]
```

이때 CPU가 Thread A를 멈추고 Thread B를 실행하려면 A의 현재 실행 상태를 저장하고 B의 이전 상태를 복원해야 합니다.

이 작업이 **Context Switch**입니다.

한 문장으로 정리하면:

> Context Switch는 CPU가 실행 중인 Process 또는 Thread의 상태를 저장하고 다른 실행 흐름의 상태를 복원해 실행 대상을 바꾸는 과정이다.

---

## 2. Context란 무엇일까?

여기서 Context는 “이 Thread가 어디까지 실행되었는가”를 다시 이어갈 수 있게 해주는 CPU 실행 상태를 뜻합니다.

대표적으로 다음 정보가 포함될 수 있습니다.

- Program Counter
- CPU Registers
- Stack Pointer
- Scheduling 관련 상태
- 필요에 따라 Memory Mapping 관련 상태

개념적으로:

```text
Thread A 실행 중
PC = 120
Register X = 5
Stack Pointer = ...

        ↓ 저장

Thread B 상태 복원
PC = 880
Register X = 42
Stack Pointer = ...

        ↓

Thread B 실행 재개
```

Thread A는 나중에 다시 실행될 때 저장된 상태에서 이어갈 수 있습니다.

---

## 3. 언제 Context Switch가 발생할까?

대표적인 경우는 다음과 같습니다.

### Time Slice가 끝났을 때

운영체제 Scheduler가 한 Thread에 일정 시간 CPU를 주고 다른 Thread에게 넘길 수 있습니다.

### 현재 Thread가 I/O를 기다릴 때

예를 들어 Disk나 Network 응답을 기다리는 동안 CPU를 계속 점유할 필요가 없습니다.

```text
Thread A
  ↓ DB 응답 대기
Blocked

CPU → Thread B 실행
```

### 더 높은 우선순위 작업이 실행 가능해졌을 때

운영체제 Scheduling 정책에 따라 다른 작업이 CPU를 받을 수 있습니다.

### Thread가 직접 양보하거나 대기 상태가 될 때

Lock, Sleep, Wait 등의 이유로 현재 Thread를 계속 실행할 수 없는 경우가 있습니다.

---

## 4. Context Switch는 왜 공짜가 아닐까?

Context Switch 중에는 애플리케이션의 실제 비즈니스 로직을 처리하지 않습니다.

즉 CPU가 “일을 바꾸기 위한 준비”를 하는 시간입니다.

대표적인 비용은 다음과 같습니다.

### CPU 상태 저장과 복원

Register와 실행 위치 등의 상태를 저장하고 새로운 상태를 복원해야 합니다.

### Scheduler 실행

다음에 실행할 Thread를 선택하는 운영체제 작업이 필요합니다.

### CPU Cache 영향

Thread A가 사용하던 데이터가 CPU Cache에 있었는데 Thread B가 실행되면 B가 필요한 데이터로 Cache가 채워질 수 있습니다.

나중에 A가 다시 실행되면 이전 데이터가 Cache에서 사라져 다시 가져와야 할 수 있습니다.

### TLB 영향

특히 Process가 바뀌면 Virtual Address Translation 관련 캐시인 TLB의 효율에도 영향을 줄 수 있습니다.

현대 CPU와 운영체제는 이를 줄이기 위한 다양한 최적화를 사용하므로 “Context Switch마다 TLB 전체가 무조건 비워진다”라고 단정하면 안 됩니다.

---

## 5. Process Switch와 Thread Switch

같은 Process 안의 Thread끼리 전환하는 경우와 서로 다른 Process 사이를 전환하는 경우 비용 구조가 완전히 같지는 않습니다.

### 같은 Process의 Thread 전환

같은 주소 공간을 공유하므로 메모리 매핑 변경 부담이 상대적으로 적을 수 있습니다.

```text
Process A
Thread 1 → Thread 2
```

### 다른 Process로 전환

주소 공간이 달라지므로 메모리 관리 관점에서 더 많은 상태 변화가 필요할 수 있습니다.

```text
Process A Thread 1
        ↓
Process B Thread 1
```

따라서 일반적으로 Process 간 전환이 같은 Process 내 Thread 전환보다 더 무거울 수 있지만, 실제 비용은 OS와 CPU 구조에 따라 달라집니다.

---

## 6. Mode Switch와 다시 구분하기

이전 문서에서 다뤘던 핵심 구분입니다.

### Mode Switch

같은 Thread가 User Mode에서 Kernel Mode로 들어갔다가 돌아올 수 있습니다.

```text
Thread A
User Mode → Kernel Mode → User Mode
```

### Context Switch

실행 주체 자체가 바뀝니다.

```text
Thread A → Thread B
```

따라서:

> System Call이 발생했다고 해서 반드시 Context Switch가 발생하는 것은 아니다.

예를 들어 System Call이 즉시 처리되어 같은 Thread로 돌아오면 Mode Switch만 있었을 수 있습니다.

반대로 I/O를 기다려야 한다면 해당 Thread가 Block되고 다른 Thread로 Context Switch될 수 있습니다.

---

## 7. Thread를 많이 만들면 왜 문제가 될까?

Thread가 많아지면 운영체제 입장에서는 실행 후보가 많아집니다.

```text
4 Core CPU

Runnable Threads = 4     → 비교적 단순
Runnable Threads = 400   → Scheduling과 전환 증가 가능
```

특히 CPU-bound 작업에서는 실제 Core 수보다 훨씬 많은 Runnable Thread가 있으면 Context Switch가 증가하면서 효율이 떨어질 수 있습니다.

비용은 다음과 같이 누적될 수 있습니다.

- Scheduler overhead
- Register save/restore
- Cache miss 증가
- Lock contention
- Thread Stack 메모리

그래서 “Thread를 더 만들면 더 빠르다”는 접근은 위험합니다.

---

## 8. 백엔드 서버에서의 예시

동기식 서버에서 많은 요청이 DB I/O를 기다린다고 생각해봅시다.

```text
Thread 1 → DB wait
Thread 2 → DB wait
Thread 3 → DB wait
Thread 4 → DB wait
...
```

요청마다 Thread를 계속 추가하면 동시 요청을 처리할 수는 있지만 Thread 수가 너무 커질 수 있습니다.

그래서 현대 백엔드 Runtime은 다음 전략을 조합합니다.

- Thread Pool
- Async I/O
- Non-blocking I/O
- Queue
- Backpressure

핵심은 **CPU를 실제로 사용할 필요가 없는 I/O 대기 시간 동안 Thread를 불필요하게 붙잡지 않는 것**입니다.

---

## 9. C# async/await와 연결

C#에서 `async/await`의 중요한 장점 중 하나는 I/O 대기 중 Thread를 계속 점유하지 않는 코드를 작성할 수 있다는 점입니다.

개념적으로:

```text
Thread
  ↓
HTTP/DB I/O 요청
  ↓
await
  ↓
Thread는 다른 작업 처리 가능
  ↓
I/O 완료
  ↓
Continuation 실행
```

하지만 다음처럼 단순화하면 안 됩니다.

> async/await = Context Switch 제거

아닙니다.

비동기 처리에도 Scheduling과 Runtime 비용이 존재하고, Continuation이 다른 Thread에서 실행될 수도 있습니다.

핵심은 **I/O 대기 동안 Thread를 Block하지 않아 Thread 자원을 더 효율적으로 사용할 수 있다**는 것입니다.

---

## 10. CPU-bound와 I/O-bound

### CPU-bound

CPU 계산 자체가 병목입니다.

예:

- 이미지 인코딩
- 압축
- 암호화
- 대규모 계산

Thread를 무작정 늘리면 Core 경쟁과 Context Switch만 증가할 수 있습니다.

### I/O-bound

Disk, Network, DB 응답 대기가 병목입니다.

예:

- API 호출
- DB Query
- 파일 읽기
- 네트워크 통신

이 경우 Async/Non-blocking 방식으로 Thread 점유를 줄이는 것이 중요할 수 있습니다.

---

## 11. TIFF-to-PDF 사례와 연결

TIFF 페이지 변환에서 `ReadRGBAImage`, Bitmap 처리처럼 CPU 연산이 큰 부분과 파일 저장/읽기처럼 I/O 성격이 있는 부분을 구분해야 합니다.

만약 CPU-bound 변환 작업을 동시에 너무 많이 실행하면:

- CPU Core 경쟁
- Context Switching 증가
- Cache 효율 저하
- Peak Memory 증가

가 생길 수 있습니다.

따라서 Parallelism을 적용할 때는 단순히 Thread 수를 늘리는 것이 아니라 실제 CPU Core 수와 작업 특성을 확인해야 합니다.

성능 최적화의 질문은 다음이어야 합니다.

> 이 작업은 CPU를 기다리는가, I/O를 기다리는가?

이 구분이 Thread와 Async 전략을 결정하는 출발점입니다.

---

## 12. 자주 하는 오해

### “Context Switch는 무조건 나쁘다”

아닙니다. 여러 프로그램을 공정하게 실행하고 I/O 대기 중 다른 작업을 처리하려면 필수적입니다. 문제는 불필요하게 과도한 Context Switching입니다.

### “System Call = Context Switch”

아닙니다. Mode Switch와 Context Switch는 다른 개념입니다.

### “Thread Switch는 비용이 없다”

같은 Process 안에서도 Register 저장/복원과 Cache 영향 등의 비용이 있습니다.

### “async/await를 쓰면 Thread가 필요 없다”

아닙니다. 애플리케이션 코드를 실제로 실행하는 시점에는 CPU와 실행 Thread가 필요합니다. I/O 대기 동안 Thread를 붙잡지 않는 것이 핵심입니다.

---

## 13. 60초 면접 답변

> Context Switch는 CPU가 현재 실행 중인 Thread나 Process의 실행 상태를 저장하고 다른 실행 흐름의 상태를 복원해 실행 대상을 바꾸는 과정입니다. Program Counter, Register, Stack Pointer 같은 상태를 저장해야 하고 Scheduler도 실행되므로 비용이 발생합니다. 또한 CPU Cache나 TLB 효율에도 영향을 줄 수 있습니다. Context Switch는 System Call에서 발생하는 User/Kernel Mode 전환과 다른 개념이며, System Call이 항상 Context Switch를 일으키는 것은 아닙니다. 백엔드에서는 Runnable Thread가 지나치게 많으면 Context Switching과 메모리 비용이 증가하므로 Thread Pool이나 Async I/O로 동시성을 제어하는 것이 중요합니다.

---

## 핵심 요약

```text
Context Switch
= 현재 실행 상태 저장
+ 다음 실행 상태 복원
+ Scheduler의 실행 대상 전환

비용
= CPU state + scheduling + cache/TLB 영향

Mode Switch ≠ Context Switch
Thread 많음 ≠ 무조건 빠름
```

다음 주제:

- CPU Scheduling
- Blocking / Non-blocking
- Concurrency와 Parallelism
