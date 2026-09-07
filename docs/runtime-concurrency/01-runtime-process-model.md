# 01. Runtime Process Model

## 한 줄 정의

**Runtime은 OS 프로세스 위에서 언어 코드가 실행되도록 메모리 관리, 스레드, 예외, JIT/인터프리팅 같은 기능을 제공하는 실행 환경이다.**

C#에서는 대표적으로 CLR(.NET Runtime)이 이 역할을 한다.

---

## 파인만식 설명

OS는 "프로세스와 스레드를 만들어 줄게"라고 한다.

Runtime은 그 위에서 이렇게 말한다.

> "너는 C# 코드만 작성해. 내가 객체를 메모리에 올리고, 메서드를 실행하고, 예외를 처리하고, 필요하면 코드를 기계어로 바꾸고, GC도 돌릴게."

즉:

```text
Application Code
    ↓
.NET Runtime / CLR
    ↓
OS Process / Thread / Virtual Memory
    ↓
CPU / RAM / Device
```

## Process와 Runtime의 관계

보통 하나의 .NET 애플리케이션은 하나의 OS Process 안에서 실행된다.

그 Process 안에는:

- managed heap
- thread pool
- loaded assemblies
- JIT compiled code
- GC state
- application threads

등이 존재한다.

Runtime 자체가 OS를 대체하는 것은 아니다.

예를 들어:

- Process 생성 → OS
- Virtual Memory → OS
- TCP Socket → OS Kernel
- C# object allocation → CLR managed heap
- Garbage Collection → CLR
- `Task` scheduling → .NET runtime abstractions

## Managed Code

C# 코드는 보통 직접 CPU machine code로 바로 배포되는 것이 아니라 IL(Intermediate Language)을 포함한 assembly 형태로 배포된다.

실행 시 Runtime이 필요에 따라 JIT compile하여 native code로 실행한다.

```text
C# Source
  ↓
IL + Metadata
  ↓
CLR / JIT
  ↓
Native Machine Code
```

AOT도 존재하므로 ".NET은 항상 JIT"라고 단정하면 안 된다.

## Managed Heap과 OS Virtual Memory

`new User()`를 호출한다고 OS에 매번 메모리를 달라고 syscall하는 것은 아니다.

CLR이 OS에서 확보한 virtual memory 범위 안에서 managed heap을 운영하고 객체를 빠르게 할당한다.

즉:

```text
OS Virtual Memory
└── .NET Process
    ├── native/runtime memory
    ├── stacks
    └── managed heap
        ├── User object
        ├── byte[]
        └── List<T>
```

## Backend에서 중요한 이유

ASP.NET Core 요청을 볼 때 단순히 "HTTP 요청이 Controller를 호출한다" 수준에서 멈추면 부족하다.

실제로는:

```text
Socket event
→ .NET runtime
→ ThreadPool / async continuation
→ Application code
→ managed allocation
→ DB/HTTP I/O
→ continuation
→ response
```

이 흐름을 알아야 다음 문제를 설명할 수 있다.

- GC pause
- ThreadPool starvation
- sync-over-async
- excessive allocation
- blocking I/O
- memory leak처럼 보이는 managed retention

## OS와 Runtime을 혼동하면 안 되는 것

| 질문 | 주 담당 |
|---|---|
| 프로세스를 누가 스케줄링? | OS |
| 스레드를 CPU core에 누가 배치? | OS Scheduler |
| C# 객체 수명 관리? | CLR GC |
| `Task` continuation scheduling? | .NET Runtime / TaskScheduler |
| Socket packet 처리? | Kernel networking stack |
| `await` 이후 코드 재개? | Compiler-generated state machine + runtime scheduling |

## 60초 면접 답변

> .NET Runtime은 OS 프로세스 위에서 C# 코드를 실행하는 관리 계층입니다. OS가 프로세스, 스레드, 가상 메모리, 소켓 같은 저수준 자원을 제공한다면 CLR은 IL 실행과 JIT/AOT, managed heap, GC, exception handling, ThreadPool 같은 기능을 제공합니다. 그래서 백엔드 장애를 볼 때 OS 자원 문제와 Runtime 문제를 구분해야 합니다. 예를 들어 CPU scheduling은 OS 영역이지만 ThreadPool starvation이나 GC pressure는 .NET Runtime 관점에서 봐야 합니다.

## 핵심 오해

- Runtime = OS가 아니다.
- `new`가 매번 OS syscall을 의미하지 않는다.
- .NET은 항상 JIT만 사용하는 것은 아니다.
- `Task`는 OS Thread와 동일하지 않다.
