# 07. Blocking / Non-blocking / Synchronous / Asynchronous

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 자신의 말로 설명할 수 있어야 합니다.

- Blocking과 Non-blocking은 무엇을 기준으로 나누는가?
- Synchronous와 Asynchronous는 무엇을 기준으로 나누는가?
- 왜 Blocking = Synchronous, Non-blocking = Asynchronous라고 단순화하면 안 되는가?
- I/O 대기 중 Thread가 어떤 상태가 되는가?
- 백엔드에서 async I/O가 Thread 사용량을 줄일 수 있는 이유는 무엇인가?
- C# `async/await`가 새 Thread를 자동으로 만드는 기능이 아닌 이유는 무엇인가?

---

## 1. 먼저 두 축을 분리해야 한다

가장 흔한 실수는 다음처럼 외우는 것입니다.

```text
Blocking = Sync
Non-blocking = Async
```

이렇게 외우면 기술면접에서 쉽게 꼬입니다.

두 개념은 질문하는 기준이 다릅니다.

### Blocking / Non-blocking

> 호출한 실행 흐름이 결과를 기다리며 멈추는가?

### Synchronous / Asynchronous

> 작업 완료를 누가 어떤 방식으로 확인하고 이어서 처리하는가?

즉 하나는 **호출자의 실행이 막히는지**, 다른 하나는 **완료 전달 방식**에 더 가깝습니다.

---

## 2. Blocking

Blocking 호출에서는 필요한 작업이 끝날 때까지 호출한 Thread가 진행하지 못할 수 있습니다.

```text
Thread
  |
  | read()
  v
[ I/O 기다림 ]
  |
  v
계속 실행
```

예를 들어 파일이나 Socket에서 데이터가 올 때까지 기다리는 동안 Thread가
Waiting 상태에 들어가면 Scheduler는 다른 Runnable Thread를 실행할 수 있습니다.

중요한 점은 **CPU가 멈추는 것이 아니라 해당 Thread가 기다린다**는 것입니다.

---

## 3. Non-blocking

Non-blocking 호출은 지금 바로 결과를 얻을 수 없더라도 호출자를 오래 붙잡지 않고
즉시 제어권을 돌려주는 방식입니다.

개념적으로:

```text
result = try_read()

if result == NOT_READY:
    다른 일 수행
```

호출자는 데이터가 준비되지 않았다는 결과를 받고 다른 작업을 할 수 있습니다.

하지만 계속 반복해서 상태를 묻는 Busy Polling을 하면 CPU를 낭비할 수 있습니다.
따라서 실제 시스템에서는 이벤트 통지 메커니즘과 함께 쓰는 경우가 많습니다.

---

## 4. Synchronous

Synchronous 방식에서는 호출 흐름이 작업의 완료 순서를 직접 따라가는 형태가
일반적입니다.

```text
A 호출
↓
A 완료
↓
B 호출
↓
B 완료
```

코드 흐름이 직관적이지만 느린 I/O를 동기적으로 기다리는 Thread가 많아지면
Thread 수가 증가할 수 있습니다.

---

## 5. Asynchronous

Asynchronous 방식에서는 작업을 시작한 뒤 완료를 나중에 전달받고 이어서 처리할
수 있습니다.

```text
I/O 시작
↓
호출자는 다른 작업 가능
↓
I/O 완료 이벤트
↓
Continuation 실행
```

완료를 전달하는 방식은 Callback, Future/Promise, Task, Event Loop 등 여러 형태가
있습니다.

---

## 6. 조합을 생각해보자

개념을 완전히 같은 축으로 보면 안 되기 때문에 다양한 조합을 생각할 수 있습니다.

### Blocking + Synchronous

가장 직관적인 형태입니다.

```text
result = read()
// 완료될 때까지 현재 흐름 대기
use(result)
```

### Non-blocking + 반복 확인

```text
while True:
    result = try_read()
    if ready(result):
        break
    do_other_work()
```

호출은 막히지 않지만 완료 여부를 호출자가 계속 확인합니다.

### Asynchronous completion

```text
start_read(on_complete)
// 다른 작업
```

작업이 끝났을 때 Runtime이나 OS 이벤트 메커니즘이 완료 사실을 전달합니다.

---

## 7. 백엔드 서버에서 Blocking I/O 문제

요청 1개당 Thread 1개가 Blocking I/O를 기다린다고 가정합니다.

```text
Request A → Thread A → DB wait
Request B → Thread B → API wait
Request C → Thread C → Disk wait
```

각 Thread는 CPU 계산을 거의 하지 않으면서 기다릴 수 있습니다.
동시 요청이 매우 많아지면 다음 문제가 생길 수 있습니다.

- 많은 Thread Stack 메모리
- Thread Pool 고갈
- Scheduling 부담
- Context Switching 증가
- 새로운 요청이 실행할 Thread를 얻지 못함

---

## 8. Async I/O의 핵심 이점

Async I/O의 핵심은 “무조건 더 빠른 I/O”가 아닙니다.

DB나 네트워크 자체가 100ms 걸리는 작업이라면 async로 바꾼다고 그 물리적
대기 시간이 1ms가 되는 것은 아닙니다.

대신:

```text
Thread
  ↓ I/O 요청
Kernel / Runtime이 완료 대기 관리
  ↓
Thread는 다른 요청 처리 가능
  ↓
I/O 완료
  ↓
Continuation 예약
```

즉 **I/O를 기다리는 동안 Thread를 붙잡지 않아 높은 동시성을 더 적은 Thread로
처리할 수 있는 것**이 중요한 이점입니다.

---

## 9. C# async/await

다음 코드를 생각해봅시다.

```csharp
var response = await httpClient.GetAsync(url);
```

`await`는 “새 Thread를 하나 만들어 기다려라”라는 뜻이 아닙니다.

I/O가 아직 끝나지 않았다면 현재 메서드의 나머지 실행을 Continuation 형태로
남기고 호출자에게 제어권을 돌려줄 수 있습니다. I/O가 완료되면 Continuation이
다시 Scheduling됩니다.

따라서:

> async/await는 Thread 생성 문법이 아니라 비동기 작업의 Continuation을 읽기
> 쉬운 코드 형태로 표현하는 언어 기능이다.

CPU-bound 작업을 단순히 `async`라고 선언한다고 계산 자체가 병렬화되지는 않습니다.

---

## 10. Event Loop와 I/O Multiplexing

많은 네트워크 연결을 적은 Thread로 다루기 위해 운영체제의 이벤트 통지 기능을
사용할 수 있습니다.

Linux에서는 `epoll`, BSD/macOS 계열에서는 `kqueue`, Windows에서는 IOCP 같은
메커니즘이 대표적입니다.

개념적으로:

```text
많은 sockets
   ↓
OS가 readiness/completion 관리
   ↓
이벤트 발생한 작업만 처리
```

이 구조 덕분에 연결마다 항상 전용 Thread 하나를 붙여둘 필요가 없습니다.

---

## 11. 언제 Blocking이 나쁜가?

Blocking 자체가 악은 아닙니다.

다음 경우에는 단순한 Blocking 코드가 충분히 합리적일 수 있습니다.

- 동시 요청 수가 작다.
- 구현 단순성이 더 중요하다.
- CPU-bound 작업이다.
- 별도 Worker Thread에서 의도적으로 Blocking한다.

반대로 대량의 네트워크 I/O를 처리하는 서버에서 Thread를 오래 붙잡는 Blocking은
확장성 병목이 되기 쉽습니다.

---

## 12. 자주 하는 오해

### “async는 코드를 더 빠르게 실행한다”

항상 그렇지 않습니다. 주로 I/O 대기 동안 Thread 자원을 더 효율적으로 활용하게
해줍니다.

### “await를 쓰면 새 Thread가 생성된다”

아닙니다. I/O async에서는 대기 전용 Thread가 필요하지 않을 수 있습니다.

### “Non-blocking이면 CPU를 전혀 쓰지 않는다”

Busy Polling처럼 잘못 구현하면 오히려 CPU를 많이 쓸 수 있습니다.

### “CPU-bound 작업도 async로 감싸면 빨라진다”

CPU 계산량은 그대로입니다. 병렬 계산이 필요하면 Core와 Thread Pool 등을 별도로
고려해야 합니다.

---

## 13. 60초 면접 답변

> Blocking과 Non-blocking은 호출한 실행 흐름이 결과를 기다리며 멈추는지를
> 기준으로 구분하고, Synchronous와 Asynchronous는 작업 완료를 어떤 방식으로
> 이어받는지를 기준으로 보는 개념입니다. 그래서 두 쌍을 완전히 같은 의미로
> 보면 안 됩니다. 백엔드에서는 Blocking I/O를 요청마다 Thread 하나가 기다리게
> 하면 Thread Pool 고갈과 Context Switch 문제가 생길 수 있습니다. Async I/O는
> I/O 자체를 마법처럼 빠르게 만드는 것이 아니라 대기하는 동안 Thread를 다른
> 요청에 활용할 수 있게 해 높은 동시성을 적은 Thread로 처리하는 데 유리합니다.
> C#의 async/await도 새 Thread를 만드는 문법이 아니라 비동기 Continuation을
> 표현하는 문법입니다.

---

## 핵심 요약

```text
Blocking      = 호출자가 기다리며 멈추는가?
Non-blocking  = 즉시 제어권을 돌려받는가?
Async         = 완료를 나중에 이어받는 구조
async/await   ≠ 새 Thread 생성
Async I/O     = I/O 속도 증가보다 Thread 효율 개선
```

다음 주제:

- Concurrency vs Parallelism
