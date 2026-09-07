# Runtime & Concurrency Mock Interview — 40 Questions

상태: 답변 대기

목표: Runtime & Concurrency 15개 주제를 자료 없이 설명하고 실제 장애 상황에 적용할 수 있는지 확인합니다.

## Part A — Fundamentals

1. CLR과 OS의 역할 차이를 설명하세요.
2. `new`를 호출할 때마다 OS syscall이 발생하나요? 이유를 설명하세요.
3. ThreadPool은 왜 존재하나요?
4. Task와 Thread의 차이를 설명하세요.
5. I/O-bound Task가 기다리는 동안 전용 Thread를 계속 점유하지 않을 수 있는 이유를 설명하세요.
6. `async/await`가 새 Thread를 만든다는 설명이 왜 틀렸나요?
7. await한 Task가 아직 완료되지 않았을 때 내부적으로 어떤 일이 일어나는지 설명하세요.
8. `.Result`와 `.Wait()`이 서버에서 위험할 수 있는 이유를 설명하세요.

## Part B — Synchronization

9. `lock`, `SemaphoreSlim`, `Interlocked`를 각각 언제 사용하나요?
10. `volatile`이 `counter++`를 thread-safe하게 만들지 못하는 이유는 무엇인가요?
11. ConcurrentDictionary를 사용하면 모든 business race condition이 해결되나요?
12. `ConcurrentDictionary.GetOrAdd`의 value factory에 중복 불가 side effect를 넣으면 위험한 이유는 무엇인가요?
13. Process 내부 lock이 여러 서버 Instance 사이의 중복 처리를 막지 못하는 이유를 설명하세요.
14. Thread-safe와 Distributed-safe를 비교하세요.

## Part C — Cancellation / Async Flow

15. Timeout과 Cancellation의 차이를 설명하세요.
16. HTTP request가 취소됐는데 downstream DB query가 계속 실행될 수 있는 이유는 무엇인가요?
17. CancellationToken은 왜 cooperative cancellation이라고 하나요?
18. Cancellation이 이미 발생한 DB commit을 rollback하나요?
19. `IAsyncEnumerable<T>`를 일반 `List<T>` 반환과 비교해 설명하세요.
20. `Channel<T>`이 producer-consumer 구조에서 유용한 이유는 무엇인가요?
21. Bounded Channel이 backpressure를 만드는 과정을 설명하세요.

## Part D — GC / Memory

22. .NET GC의 Gen 0, 1, 2가 존재하는 이유를 설명하세요.
23. GC Root와 reachability를 설명하세요.
24. LOH는 무엇이며 백엔드에서 왜 신경 써야 하나요?
25. Heap size가 안정적인데도 GC CPU가 높을 수 있는 이유는 무엇인가요?
26. Boxing이 무엇이고 hot path에서 왜 문제가 될 수 있나요?
27. Closure, LINQ, string 생성이 allocation pressure를 만들 수 있는 이유를 설명하세요.
28. GC가 있는데도 managed memory leak이 가능한 이유를 설명하세요.
29. static collection, event handler, timer가 memory leak을 만드는 방식을 설명하세요.
30. RSS와 managed heap의 차이를 설명하세요.

## Part E — Resource Lifetime / Context

31. GC와 `IDisposable`의 역할을 비교하세요.
32. `using`과 `await using`의 차이를 설명하세요.
33. DB Connection, Stream, Socket 같은 resource를 늦게 dispose하면 어떤 문제가 발생할 수 있나요?
34. ExecutionContext와 SynchronizationContext를 비교하세요.
35. `AsyncLocal<T>`과 tracing context가 async boundary를 넘어 전달될 수 있는 이유를 설명하세요.

## Part F — Diagnostics / Scenarios

36. CPU가 40%인데 API p99 latency가 크게 증가했습니다. ThreadPool starvation을 어떻게 확인하겠습니까?
37. ThreadPool starvation과 CPU saturation을 어떻게 구분하겠습니까?
38. 프로세스 RSS가 계속 증가합니다. 어떤 지표와 dump를 어떤 순서로 보겠습니까?
39. GC CPU가 높고 latency spike가 있습니다. 어떤 allocation 원인을 조사하겠습니까?
40. 요청 취소 후에도 DB와 downstream HTTP 호출이 계속 쌓입니다. 원인 가설과 개선책을 설명하세요.

---

## 평가 기준

각 문항을 다음 기준으로 평가합니다.

- 2점: 핵심 개념 + 인과관계 + backend 연결까지 정확
- 1점: 핵심은 맞지만 trade-off/원인 설명 부족
- 0점: 핵심 개념 혼동

총점 80점.

### Review 통과 기준

- 최소 64/80점
- 40문항 중 최소 32문항 핵심 혼동 없음
- 다음 비교는 반드시 통과:
  - Thread vs Task
  - Blocking vs async suspension
  - lock vs SemaphoreSlim
  - Timeout vs Cancellation
  - GC vs Dispose
  - ThreadPool Starvation vs CPU Saturation
- 장애 시나리오에서 `증상 → 지표 → 원인 가설 → 검증 → 조치` 순서가 포함되어야 함

D+1 재시험 전에는 Completed로 변경하지 않습니다.
