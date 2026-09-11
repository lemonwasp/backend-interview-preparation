# TaskScheduler and Work-Stealing

## 한 줄 정의

**Work-stealing은 여러 worker thread가 각자 local queue를 처리하다가 일이 없는 worker가 다른 worker의 queue에서 작업을 가져오는 스케줄링 전략이다. .NET ThreadPool과 Task 실행 모델을 이해할 때 중요한 개념이다.**

---

## 1. 왜 필요한가

멀티코어 시스템에서 여러 worker가 동시에 일을 처리한다고 하자.

```text
Worker A -> 20 jobs
Worker B -> 0 jobs
Worker C -> 3 jobs
Worker D -> 0 jobs
```

단순히 각 worker가 자기 queue만 처리하면 B와 D는 놀고 A는 과부하된다.

Work-stealing에서는 idle worker가 다른 worker의 작업 일부를 가져온다.

```text
Worker B steals from A
Worker D steals from A
```

결과적으로 load가 더 고르게 분산된다.

---

## 2. Local Queue와 Global Queue

개념적으로는 두 종류의 queue를 생각할 수 있다.

```text
Global Queue
    |
    +--> Worker A Local Queue
    +--> Worker B Local Queue
    +--> Worker C Local Queue
```

보통 worker가 스스로 생성한 작업은 local queue에 넣을 수 있고,
외부에서 들어온 작업이나 공용 작업은 global queue에 들어갈 수 있다.

local queue의 장점은 contention을 줄이는 것이다.

모든 worker가 하나의 global queue만 두고 경쟁하면:

```text
Worker A --\
Worker B ----> one shared queue
Worker C --/
```

queue lock이나 cache contention이 커질 수 있다.

---

## 3. 왜 deque가 자주 등장하나

Work-stealing 구조에서는 double-ended queue(deque)가 자주 사용된다.

owner worker는 한쪽 끝에서 작업을 꺼내고,
stealer는 반대쪽 끝에서 작업을 가져간다.

```text
Owner -> [task][task][task][task] <- Stealer
```

이렇게 하면 owner와 stealer가 같은 위치를 두고 경쟁하는 빈도를 줄일 수 있다.

---

## 4. Task와 Thread는 같은 것이 아니다

```csharp
Task.Run(() => DoWork());
```

를 호출했다고 새로운 Thread가 반드시 하나 생성되는 것은 아니다.

`Task`는 작업의 추상화이고,
실제 실행은 일반적으로 ThreadPool worker가 담당한다.

```text
Task
  ↓
TaskScheduler
  ↓
ThreadPool
  ↓
Worker Thread
```

따라서:

```text
1000 Tasks != 1000 Threads
```

이다.

---

## 5. TaskScheduler

.NET의 `TaskScheduler`는 Task가 어떻게 실행될지를 조정하는 abstraction이다.

일반적으로 기본 scheduler는 ThreadPool을 사용한다.

```csharp
Task.Run(() => Work());
```

대부분의 서버 코드에서는 기본 scheduler를 직접 교체할 필요가 없다.

하지만 개념을 이해하면 다음 문제를 설명하기 쉬워진다.

- 왜 Task가 바로 실행되지 않을 수 있는가
- 왜 ThreadPool starvation이 latency를 만든는가
- 왜 CPU-bound 작업을 무한히 Task.Run 하면 안 되는가

---

## 6. CPU-bound 작업과 Queue

CPU core가 8개인데 CPU-bound Task를 10,000개 만든다고 하자.

```text
10000 tasks
    ↓
ThreadPool queues
    ↓
limited CPU cores
```

Task 수를 늘린다고 CPU 처리량이 선형으로 증가하지 않는다.

오히려:

- scheduling overhead
- context switching
- cache pollution
- queue latency

가 증가할 수 있다.

---

## 7. I/O-bound 작업과 차이

I/O-bound async 작업은 I/O 대기 중 worker를 계속 점유하지 않을 수 있다.

```text
Request
  ↓
await DB/network
  ↓
Thread returned to pool
  ↓
I/O completion
  ↓
continuation scheduled
```

반면 CPU-bound 작업은 실제 CPU 시간을 계속 소비한다.

```text
CPU-bound -> worker + CPU 필요
I/O-bound  -> 대기 중 worker를 놓을 수 있음
```

이 차이는 백엔드 성능 면접에서 매우 중요하다.

---

## 8. Work-Stealing의 장점

### Load balancing

한 worker에 작업이 몰려도 idle worker가 가져갈 수 있다.

### Locality

자기 local queue의 작업을 계속 처리하면 shared global queue 접근을 줄일 수 있다.

### Scalability

worker 수가 증가할 때 하나의 중앙 queue에 대한 contention을 줄이는 데 도움이 된다.

---

## 9. Work-Stealing의 비용

무료는 아니다.

stealing 자체에도 synchronization과 탐색 비용이 있다.

또한 작은 작업을 지나치게 잘게 나누면:

```text
actual work = 2 us
scheduling overhead = significant
```

처럼 작업보다 scheduling 비용이 커질 수 있다.

따라서 parallelism에서는 작업 granularity가 중요하다.

---

## 10. Long-running 작업

매우 오래 blocking되는 작업을 ThreadPool에 무분별하게 넣으면 pool worker를 오래 점유할 수 있다.

```text
ThreadPool worker
   ↓
blocking for minutes
   ↓
available worker 감소
```

이런 workload는 일반적인 짧은 ThreadPool 작업과 다른 설계가 필요할 수 있다.

핵심은 `Task.Run`을 "아무 작업이나 백그라운드로 보내는 마법"처럼 사용하지 않는 것이다.

---

## 11. ThreadPool Starvation과 연결

worker가 모두 blocking되면 새 Task나 continuation이 실행되기 어려워진다.

```text
workers blocked
      ↓
queue grows
      ↓
continuations delayed
      ↓
request latency increases
```

이 때문에 sync-over-async, 긴 blocking I/O, 과도한 CPU-bound Task는 ThreadPool 관점에서 함께 봐야 한다.

---

## 12. 면접 질문

### Q. Task와 Thread의 차이는 무엇인가요?

> Task는 비동기 작업과 완료 상태를 표현하는 abstraction이고, Thread는 실제 실행 단위입니다. 많은 Task가 소수의 ThreadPool worker 위에서 스케줄될 수 있으므로 Task 하나당 Thread 하나가 생성되는 것은 아닙니다.

### Q. Work-stealing이 필요한 이유는 무엇인가요?

> worker별 local queue를 사용하면 중앙 queue contention을 줄일 수 있지만 작업 분배가 불균형해질 수 있습니다. idle worker가 다른 worker의 queue에서 작업을 가져오면 load balancing을 개선할 수 있습니다.

### Q. CPU-bound 작업을 Task.Run으로 많이 만들면 왜 문제가 되나요?

> 실제 CPU core 수는 제한되어 있기 때문에 Task 수를 늘려도 처리량이 계속 증가하지 않습니다. 오히려 queueing, scheduling, context switching 비용이 커지고 다른 요청의 continuation까지 지연시킬 수 있습니다.

---

## 13. 60초 답변

> .NET에서 Task는 Thread 자체가 아니라 실행할 작업의 abstraction이고 기본적으로 TaskScheduler와 ThreadPool을 통해 실행됩니다. ThreadPool은 여러 worker가 작업을 처리하며, work-stealing 같은 방식으로 worker별 local queue의 불균형을 완화할 수 있습니다. Local queue는 중앙 queue contention을 줄이는 데 유리하고, idle worker는 다른 worker의 작업을 가져와 CPU 활용률을 높일 수 있습니다. 하지만 CPU-bound Task를 과도하게 만들면 실제 core 수보다 일이 많아져 scheduling overhead와 queue latency가 증가하므로 Task 수와 병렬성은 구분해서 생각해야 합니다.

## 기억할 핵심

```text
1. Task != Thread
2. 많은 Task가 소수 ThreadPool worker에서 실행될 수 있다.
3. Local queue는 shared contention을 줄인다.
4. Work-stealing은 worker 간 load imbalance를 줄인다.
5. CPU-bound 작업은 Task 수보다 실제 core 수가 중요하다.
```
