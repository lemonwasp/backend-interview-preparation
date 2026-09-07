# 14. I/O Multiplexing

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 설명할 수 있어야 합니다.

- 수천 개 Socket마다 Thread 하나를 붙이는 방식의 한계는 무엇인가?
- I/O Multiplexing은 무엇을 해결하는가?
- `select`, `poll`, `epoll`, `kqueue`, IOCP는 어떤 계열의 문제를 해결하는가?
- Readiness 기반 모델과 Completion 기반 모델의 차이는 무엇인가?
- Event Loop와 `async/await`는 OS I/O 모델과 어떻게 연결되는가?

---

## 1. Thread-per-connection의 문제

가장 단순한 서버 모델은 Connection마다 Thread 하나를 붙이는 것입니다.

```text
Connection 1 → Thread 1
Connection 2 → Thread 2
Connection 3 → Thread 3
...
```

Connection 수가 적으면 이해하기 쉽습니다.

하지만 수천, 수만 Connection이 대부분 Network I/O를 기다리고 있다면 많은 Thread가 실제 계산 대신 대기하게 됩니다.

문제는:

- Thread Stack Memory
- Scheduling 비용
- Context Switch 증가
- Thread Pool 고갈
- 운영 복잡성

등입니다.

---

## 2. 핵심 아이디어

Network Socket 여러 개 중 **지금 읽거나 쓸 준비가 된 것만 알려달라**고 Kernel에 요청하면 어떨까요?

```text
Socket A ─ waiting
Socket B ─ ready
Socket C ─ waiting
Socket D ─ ready
       ↓
Kernel
       ↓
"B와 D가 준비됨"
```

애플리케이션은 준비된 Socket만 처리할 수 있습니다.

이것이 I/O Multiplexing의 핵심 아이디어입니다.

> 적은 수의 Thread로 많은 I/O 대기 대상을 관리한다.

---

## 3. select

`select`는 여러 File Descriptor의 상태를 한 번에 기다릴 수 있게 하는 오래된 인터페이스입니다.

개념적으로:

```text
fds = [1,2,3,4,5,...]
        ↓
select()
        ↓
ready = [2,5]
```

하지만 큰 Descriptor 집합을 반복해서 전달하고 검사해야 하는 비용과 개수 제한 등의 문제가 있을 수 있습니다.

---

## 4. poll

`poll`은 `select`의 일부 제약을 개선하지만 많은 Descriptor를 매번 순회해야 하는 특성은 남습니다.

Connection 수가 매우 커질수록 전체 목록을 반복 검사하는 비용이 문제가 될 수 있습니다.

---

## 5. epoll

Linux의 `epoll`은 관심 있는 Descriptor를 Kernel에 등록해두고, 준비된 Event 중심으로 가져올 수 있게 설계되어 있습니다.

개념적으로:

```text
register sockets once
        ↓
Kernel tracks events
        ↓
epoll_wait()
        ↓
return ready sockets
```

대규모 Connection 처리에서 `select/poll`보다 효율적인 이유가 여기에 있습니다.

---

## 6. kqueue

BSD/macOS 계열에서는 `kqueue`가 비슷한 목적의 Event Notification 메커니즘을 제공합니다.

네트워크 Socket뿐 아니라 여러 Kernel Event를 통합해서 감시할 수 있습니다.

---

## 7. Windows IOCP

Windows에서는 IOCP(I/O Completion Ports)가 대표적인 고성능 비동기 I/O 모델입니다.

`epoll`과 정확히 같은 모델이라고 보면 안 됩니다.

### Readiness 모델

```text
"이 Socket은 지금 읽을 수 있습니다."
```

대표적으로 `epoll`, `kqueue` 계열을 이런 관점으로 이해할 수 있습니다.

### Completion 모델

```text
"요청했던 I/O 작업이 완료되었습니다."
```

IOCP는 Completion 중심 모델로 이해하는 것이 좋습니다.

---

## 8. Level-triggered와 Edge-triggered

`epoll` 등 일부 인터페이스에서는 Event 전달 방식도 구분할 수 있습니다.

### Level-triggered

조건이 계속 만족되는 동안 Ready 상태를 계속 알려줄 수 있습니다.

### Edge-triggered

상태 변화가 발생한 시점 중심으로 알려줍니다.

Edge-triggered는 Event 수를 줄일 수 있지만 데이터를 충분히 Drain하지 않으면 다음 알림을 놓치는 식의 구현 실수가 생길 수 있습니다.

---

## 9. Event Loop

I/O Multiplexing은 Event Loop 구조와 자주 연결됩니다.

```text
while true:
    events = wait_for_ready_io()

    for event in events:
        handle(event)
```

Thread가 Socket 하나에 묶여 Blocking하는 대신 여러 Connection의 Event를 순환 처리합니다.

Node.js, Nginx, Redis 같은 시스템을 이해할 때 중요한 개념입니다.

---

## 10. C# async/await와 연결

ASP.NET Core에서 다음 코드를 쓴다고 합시다.

```csharp
var result = await httpClient.GetAsync(url);
```

Network I/O를 기다리는 동안 요청 처리 Thread가 반드시 계속 Block되어 있을 필요는 없습니다.

Runtime과 OS의 비동기 I/O 메커니즘이 작업 완료를 감지하고, 완료 후 Continuation이 다시 실행될 수 있습니다.

개념적으로:

```text
request thread
   ↓
start async I/O
   ↓
thread returned to pool
   ↓
OS waits for I/O
   ↓
I/O completes
   ↓
continuation scheduled
```

따라서 `async/await`의 중요한 가치 중 하나는 많은 I/O 대기 요청을 적은 Thread로 처리하기 쉽게 만드는 것입니다.

---

## 11. I/O Multiplexing = Parallelism은 아니다

Event Loop 하나가 수천 Connection을 관리한다고 해서 요청들이 CPU에서 실제 동시에 병렬 실행된다는 뜻은 아닙니다.

```text
Concurrency = 많은 작업의 진행을 겹쳐 관리
Parallelism = 여러 CPU Core에서 실제 동시 실행
```

I/O Multiplexing은 주로 **I/O Concurrency를 효율적으로 관리**하는 기술입니다.

---

## 12. 언제 Thread Pool이 여전히 필요한가?

I/O Multiplexing이 모든 일을 대신하지는 않습니다.

CPU-heavy 작업:

- Image Processing
- Compression
- Encryption
- Complex Serialization
- ML Inference 일부

같은 작업은 실제 CPU 시간을 소비합니다.

Event Loop Thread에서 이런 작업을 오래 수행하면 다른 Connection 처리가 막힐 수 있으므로 Worker Thread/Process Pool로 분리할 수 있습니다.

---

## 13. 백엔드 설계 예시

### 잘못된 구조

```text
10,000 connections
   ↓
10,000 blocking threads
```

### 효율적인 구조 예

```text
10,000 connections
   ↓
OS async event mechanism
   ↓
small number of I/O threads
   ↓
CPU work → worker pool
```

실제 Runtime 구현은 더 복잡하지만 핵심 원리는 이렇습니다.

---

## 14. 자주 하는 오해

### “epoll이면 모든 요청이 빨라진다”

아닙니다. I/O 대기 Connection을 효율적으로 관리하는 것이 핵심이지 CPU-heavy 작업 자체를 빠르게 만드는 기술은 아닙니다.

### “async/await는 epoll 문법이다”

아닙니다. `async/await`는 언어 수준 추상화이며 실제 OS 메커니즘은 Runtime과 플랫폼에 따라 달라집니다.

### “Event Loop에는 Thread가 없다”

아닙니다. Event Loop 자체도 CPU에서 실행되는 Thread가 필요합니다.

### “Non-blocking이면 무조건 Async다”

아닙니다. Blocking/Non-blocking과 Sync/Async는 구분해야 합니다.

---

## 15. 60초 면접 답변

> I/O Multiplexing은 많은 Socket이나 File Descriptor를 각각 Thread 하나로 기다리는 대신, Kernel이 여러 I/O 대상의 상태를 감시하고 준비된 Event만 애플리케이션에 알려주는 방식입니다. Linux에서는 epoll, BSD/macOS에서는 kqueue 같은 메커니즘이 대표적이고, Windows의 IOCP는 완료 통지 중심의 비동기 I/O 모델입니다. 이런 구조를 사용하면 수많은 Network Connection을 적은 수의 Thread로 관리할 수 있어 Thread Memory와 Context Switching 비용을 줄일 수 있습니다. 다만 CPU-heavy 작업은 별도 Worker Pool이 필요할 수 있고, I/O Multiplexing 자체가 CPU Parallelism을 만드는 것은 아닙니다.

---

## 핵심 요약

```text
Thread-per-connection → 많은 대기 Thread
I/O Multiplexing → 준비된 I/O만 Event로 처리
select/poll → 전체 집합 검사 비용
epoll/kqueue → Event 중심 readiness
IOCP → completion 중심
async/await → OS/Runtime 비동기 I/O를 쓰기 쉽게 만드는 추상화
```

다음 주제 후보:

- Socket과 TCP 연결 흐름
- Zero-copy / sendfile
- Interrupt와 DMA
- File Descriptor
