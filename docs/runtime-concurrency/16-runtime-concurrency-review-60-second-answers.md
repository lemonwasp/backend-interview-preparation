# 16. Runtime & Concurrency Review — 60초 답변 총정리

Runtime & Concurrency 트랙의 목표는 C#/.NET 백엔드 코드가 실제 실행될 때 Thread, Task, async/await, GC, Memory, Cancellation, Context, Resource Lifetime이 어떻게 상호작용하는지 설명하고 장애 상황에서 원인을 좁힐 수 있는 수준에 도달하는 것입니다.

## 1. Runtime과 OS의 관계

CLR은 OS를 대체하지 않습니다. OS가 Process, Virtual Memory, Thread, Socket, File Descriptor 같은 기반을 제공하고 CLR은 그 위에서 managed code, GC, JIT/AOT, ThreadPool, Exception, Task 같은 실행 환경을 제공합니다.

### 60초 답변

> .NET Runtime인 CLR은 OS 위에서 동작하는 managed execution environment입니다. OS가 Process와 Thread, Virtual Memory, I/O를 제공하고 CLR은 그 위에서 JIT, GC, ThreadPool, Task 같은 기능을 제공합니다. 그래서 .NET 성능 문제를 볼 때도 Runtime metric만 보지 않고 CPU, memory, I/O 같은 OS 지표를 함께 봐야 합니다.

---

## 2. ThreadPool

ThreadPool은 worker thread를 재사용해 thread 생성/종료 비용을 줄입니다. 하지만 blocking 작업이 많으면 worker가 묶여 queue가 쌓이고 ThreadPool starvation이 발생할 수 있습니다.

### 60초 답변

> ThreadPool은 매 요청마다 새 Thread를 만드는 대신 worker를 재사용하는 구조입니다. 백엔드에서는 scalability에 유리하지만 sync-over-async, blocking I/O, 긴 lock 같은 작업이 많으면 worker가 고갈되어 queue가 늘고 latency가 급증할 수 있습니다. 따라서 CPU 사용률만 보고 판단하지 않고 ThreadPool queue와 thread count를 같이 봐야 합니다.

---

## 3. Task는 Thread가 아니다

Task는 작업의 완료, 결과, 실패를 표현하는 abstraction입니다. I/O Task는 대기 중 전용 Thread를 붙잡지 않을 수 있습니다.

### 60초 답변

> C#의 Task는 Thread 자체가 아니라 비동기 작업의 완료 상태와 결과를 표현하는 abstraction입니다. CPU-bound Task는 ThreadPool에서 실행될 수 있지만 I/O-bound Task는 커널 I/O completion을 기다리는 동안 Thread를 계속 점유하지 않을 수 있습니다. 그래서 Task 수와 Thread 수를 동일하게 보면 안 됩니다.

---

## 4. async/await 내부 동작

Compiler가 async method를 state machine으로 변환합니다. 미완료 Task를 await하면 현재 실행을 중단하고 continuation을 등록한 뒤 control을 caller에게 돌려줍니다.

### 60초 답변

> async/await는 새 Thread를 만드는 문법이 아닙니다. Compiler가 method를 state machine으로 바꾸고, await한 Task가 아직 끝나지 않았다면 현재 실행을 멈추고 continuation을 등록합니다. I/O가 완료되면 continuation이 다시 실행되기 때문에 blocking 없이 많은 요청을 처리할 수 있습니다.

---

## 5. lock / SemaphoreSlim / Interlocked

- `lock`: mutual exclusion
- `SemaphoreSlim`: 동시 실행 개수 제한
- `Interlocked`: 단순 atomic operation
- `volatile`: visibility/order 의미

### 60초 답변

> lock은 하나의 critical section에 한 Thread만 들어가게 하고, SemaphoreSlim은 동시에 N개까지 허용할 수 있습니다. Interlocked는 counter 증가 같은 단순 atomic 연산에 더 가볍게 쓸 수 있습니다. volatile은 최신 값 visibility를 돕지만 counter++ 같은 compound operation을 atomic하게 만들지는 않습니다.

---

## 6. Concurrent Collection

Concurrent collection은 내부 구조의 thread safety를 제공하지만 여러 연산을 묶은 business invariant까지 atomic하게 만들지는 않습니다.

### 60초 답변

> ConcurrentDictionary 같은 컬렉션은 여러 Thread가 동시에 접근할 때 내부 자료구조가 깨지지 않도록 합니다. 하지만 '확인 후 추가 후 외부 API 호출'처럼 여러 단계의 비즈니스 로직 전체를 atomic하게 만드는 것은 아닙니다. 또 Process 내부 안전성과 여러 서버 Instance 간 distributed consistency는 별개의 문제입니다.

---

## 7. Cancellation과 Timeout

Timeout은 기다림 정책이고 Cancellation은 작업 중단 요청입니다. 둘 다 이미 발생한 side effect를 자동 rollback하지 않습니다.

### 60초 답변

> Timeout은 caller가 얼마나 기다릴지 정하는 정책이고 Cancellation은 underlying work에게 중단을 요청하는 cooperative mechanism입니다. timeout이 발생했다고 작업이 자동 종료되지는 않으므로 CancellationToken을 downstream까지 전달해야 합니다. 또한 이미 DB commit이나 외부 side effect가 발생했다면 cancellation이 rollback해주지 않습니다.

---

## 8. GC Generations

Gen 0 / 1 / 2는 대부분의 객체가 짧게 산다는 가정을 활용합니다. 오래 살아남은 객체는 higher generation으로 promotion됩니다.

### 60초 답변

> .NET GC는 대부분의 객체가 짧게 산다는 특성을 이용해 Gen 0을 자주 수집하고 오래 살아남는 객체를 Gen 1, Gen 2로 승격합니다. 큰 객체는 LOH에 들어갈 수 있고, allocation rate가 높거나 long-lived object가 많으면 GC 비용과 pause가 증가할 수 있습니다.

---

## 9. Allocation / Boxing

Heap size만큼 allocation rate도 중요합니다. Boxing, closure, string 생성, LINQ, 큰 buffer는 hot path에서 누적될 수 있습니다.

### 60초 답변

> Allocation은 개별적으로 싸더라도 hot path에서 반복되면 GC pressure를 높입니다. Value Type을 object/interface로 다룰 때 boxing이 생길 수 있고, closure나 문자열 조합, LINQ도 allocation을 만들 수 있습니다. 그래서 최적화는 반드시 profiler로 allocation hotspot을 확인한 뒤 해야 합니다.

---

## 10. Managed Memory Leak

GC가 있어도 leak은 가능합니다. 필요 없는 객체가 GC Root에서 reachable하면 수집되지 않습니다.

### 60초 답변

> Managed memory leak은 free를 안 해서가 아니라 더 이상 필요 없는 객체가 static collection, event handler, timer, cache 같은 reference 때문에 계속 reachable한 상태입니다. 진단할 때는 RSS와 managed heap을 구분하고 heap dump에서 retention path와 GC root를 확인합니다.

---

## 11. IDisposable / Resource Lifetime

GC는 managed object memory를 처리하지만 socket, file handle, DB connection 같은 resource의 deterministic release는 `Dispose`가 담당합니다.

### 60초 답변

> IDisposable은 GC와 역할이 다릅니다. GC는 managed memory를 수집하지만 파일 핸들, socket, DB connection 같은 외부 resource는 언제 해제될지 예측할 수 없으므로 using으로 명시적으로 dispose해야 합니다. 비동기 정리가 필요하면 IAsyncDisposable과 await using을 사용합니다.

---

## 12. IAsyncEnumerable / Channel

`IAsyncEnumerable<T>`는 비동기 streaming consumption, `Channel<T>`은 producer-consumer coordination에 적합합니다.

### 60초 답변

> IAsyncEnumerable은 전체 결과를 메모리에 모으지 않고 비동기로 한 항목씩 소비할 수 있게 하고, Channel은 producer와 consumer 사이의 비동기 queue 역할을 합니다. Bounded Channel을 사용하면 producer가 consumer 속도를 초과할 때 backpressure를 걸 수 있습니다.

---

## 13. ThreadPool Starvation

대표 신호:
- CPU가 100%가 아닌데 latency 급증
- ThreadPool queue 증가
- worker thread 증가
- `.Result`, `.Wait`, blocking I/O, lock wait 다수

### 60초 답변

> ThreadPool starvation은 CPU가 부족해서가 아니라 worker가 blocking되어 새 작업을 처리하지 못하는 상태입니다. queue length와 thread count가 증가하고 CPU는 의외로 높지 않을 수 있습니다. dotnet-counters와 trace로 threadpool queue, worker 수, blocking stack을 확인하고 sync-over-async나 blocking dependency를 찾습니다.

---

## 14. ExecutionContext

ExecutionContext는 logical call context를 async boundary를 넘어 흐르게 합니다. `AsyncLocal<T>`, security/context 정보 등이 여기에 관련됩니다.

### 60초 답변

> ExecutionContext는 Thread 자체가 아니라 logical execution flow에 붙는 context를 async 작업 사이에 전달합니다. AsyncLocal이나 tracing context가 대표적입니다. SynchronizationContext가 continuation을 어느 환경에서 실행할지와 관련 있다면 ExecutionContext는 어떤 logical context를 전달할지에 더 가깝습니다.

---

## 15. Runtime Observability

주요 범주:
- Request: throughput, errors, p95/p99
- ThreadPool: queue, worker count
- GC: heap, Gen 2, LOH, pause, allocation rate
- Memory: RSS vs managed heap
- Lock: contention
- CPU profile
- Trace: downstream latency

### 60초 답변

> Runtime 문제는 단일 metric으로 진단하지 않습니다. latency가 증가하면 request p95/p99, CPU, ThreadPool queue, GC pause, allocation rate, managed heap, lock contention, downstream trace를 함께 봅니다. Observability의 목적은 metric에서 증상을 보고 trace/profile로 원인 가설을 검증하는 것입니다.

---

# 반드시 구분할 비교 질문

## Thread vs Task

- Thread: OS execution resource
- Task: 작업 완료 abstraction

## Blocking vs async suspension

- Blocking: Thread가 기다림
- async suspension: Thread를 반환하고 continuation으로 재개 가능

## lock vs SemaphoreSlim

- lock: 1개
- SemaphoreSlim: N개

## Thread-safe vs Distributed-safe

- Thread-safe: 한 Process 내부
- Distributed-safe: 여러 Instance 간 coordination

## Timeout vs Cancellation

- Timeout: 기다림 제한
- Cancellation: 작업 중단 요청

## GC vs Dispose

- GC: managed memory
- Dispose: deterministic resource release

## Memory Leak vs Resource Leak

- Memory Leak: 불필요한 객체가 reachable
- Resource Leak: handle/socket/connection 등 미해제

## IAsyncEnumerable vs Channel

- IAsyncEnumerable: async stream consumption
- Channel: producer-consumer queue/coordination

## ExecutionContext vs SynchronizationContext

- ExecutionContext: logical context flow
- SynchronizationContext: continuation scheduling context

## CPU Saturation vs ThreadPool Starvation

- CPU Saturation: CPU capacity 부족
- Starvation: worker가 blocking되어 queue 처리 불가

---

# 장애 시나리오 체크포인트

## 시나리오 1 — CPU는 40%인데 API p99가 급증

확인 순서:
1. ThreadPool queue
2. worker count
3. blocking stack
4. `.Result/.Wait()`
5. DB/HTTP sync I/O
6. lock wait

## 시나리오 2 — 메모리가 계속 증가

확인 순서:
1. RSS
2. managed heap
3. Gen 2 / LOH
4. allocation rate
5. heap dump
6. retention path
7. static/event/cache/timer

## 시나리오 3 — GC CPU가 높다

확인 순서:
1. allocation rate
2. Gen 0 frequency
3. Gen 2 frequency
4. LOH
5. hot allocation stack
6. buffer/string/boxing/closure

## 시나리오 4 — 요청 취소 후에도 DB 부하가 계속됨

가능성:
- token 미전파
- DB provider가 cancellation 미지원/지연
- timeout만 걸고 underlying work는 계속 실행

## 시나리오 5 — Channel queue가 계속 증가

원인:
- producer rate > consumer rate
- unbounded channel

대응:
- bounded capacity
- concurrency 조정
- load shedding
- producer throttling

## 시나리오 6 — Socket/DB connection 고갈

확인:
- Dispose 누락
- request scope보다 긴 resource lifetime
- connection pool wait
- 잘못된 HttpClient lifecycle

---

# Runtime 트랙 완료 기준

문서를 읽은 것만으로 완료가 아닙니다.

최소 기준:
1. 40문항 Mock Interview 중 32개 이상 핵심 혼동 없이 답변
2. Thread/Task, Blocking/Async, GC/Dispose, Timeout/Cancellation 비교 질문 통과
3. Starvation, Memory Leak, GC pressure 장애 시나리오에서 `지표 → 가설 → 검증 → 조치` 순서로 설명
4. 틀린 내용을 해당 문서에 반영
5. D+1 재시험 수행

다음 트랙은 System Design입니다.
