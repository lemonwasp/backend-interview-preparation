# 03. Process와 Thread

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 자신의 말로 설명할 수 있어야 합니다.

- Process와 Thread는 무엇이 다른가?
- 왜 하나의 Process 안에 여러 Thread를 둘까?
- Thread끼리는 무엇을 공유하고 무엇을 따로 가지는가?
- Multi-process와 Multi-thread의 장단점은 무엇인가?
- 백엔드 서버에서 Thread Pool이 왜 필요한가?

---

## 1. Process는 실행 중인 프로그램이다

디스크에 저장된 실행 파일은 아직 단순한 파일입니다.
운영체제가 그것을 실행하면 메모리와 각종 자원이 할당되고 하나의 실행 단위가 됩니다.

이 실행 중인 프로그램을 **Process**라고 합니다.

```text
program.exe  --실행-->  Process
```

Process는 단순히 코드만 가진 것이 아니라 보통 다음과 같은 실행 환경을 가집니다.

- 독립적인 가상 주소 공간
- 열린 파일과 Socket 정보
- 실행 중인 Thread
- 운영체제가 관리하는 Process ID
- 권한과 자원 정보

한 문장으로 정리하면:

> Process는 운영체제가 독립된 실행 단위로 관리하는 프로그램의 실행 인스턴스다.

---

## 2. Thread는 Process 안의 실제 실행 흐름이다

Process를 하나의 회사라고 생각해봅시다.

- Process = 회사 전체
- Thread = 회사 안에서 실제 일을 하는 직원

회사가 존재한다고 해서 일이 자동으로 처리되는 것은 아닙니다.
실제 명령을 실행하는 흐름이 필요하고, 그것이 Thread입니다.

한 Process 안에는 하나 이상의 Thread가 존재할 수 있습니다.

```text
Process
├── Thread 1
├── Thread 2
└── Thread 3
```

각 Thread는 CPU에서 실행될 수 있는 독립적인 실행 흐름입니다.

---

## 3. Thread끼리는 무엇을 공유할까?

같은 Process의 Thread들은 같은 주소 공간 안에서 동작하므로 여러 자원을 공유합니다.

대표적으로 다음을 공유할 수 있습니다.

- Code 영역
- Heap
- 전역 데이터
- 열린 파일
- Socket
- Process 단위 자원

반면 각 Thread는 자신의 실행 상태를 따로 가져야 합니다.

대표적으로:

- Stack
- Program Counter
- CPU Register 상태

즉:

```text
Process
├── Shared: Code / Heap / Global Data / Files / Sockets
├── Thread A: Stack + Registers + Program Counter
└── Thread B: Stack + Registers + Program Counter
```

왜 Stack은 따로 필요할까요?

각 Thread는 서로 다른 함수를 실행하고 서로 다른 지역변수와 호출 기록을 가져야 하기 때문입니다.

---

## 4. Process끼리는 왜 기본적으로 격리되어 있을까?

Process A의 메모리 버그가 Process B의 메모리를 마음대로 덮어쓰면 시스템은 매우 불안정해집니다.

그래서 운영체제는 각 Process에 독립된 가상 주소 공간이 있는 것처럼 보이게 만듭니다.

```text
Process A address space
Process B address space
Process C address space
```

기본적으로 서로 직접 접근할 수 없습니다.

필요하면 다음과 같은 IPC(Inter-Process Communication)를 사용합니다.

- Pipe
- Socket
- Shared Memory
- Message Queue

Process 간 통신은 Thread 간 메모리 공유보다 일반적으로 더 명시적인 절차가 필요합니다.

---

## 5. Multi-process와 Multi-thread

### Multi-process

여러 Process를 띄워 일을 나누는 방식입니다.

장점:

- 메모리가 격리되어 장애 전파가 적다.
- 한 Process가 죽어도 다른 Process가 살아남을 수 있다.
- 보안 경계가 더 강하다.

단점:

- Process 생성과 관리 비용이 더 크다.
- 메모리를 공유하기 어렵다.
- IPC가 필요할 수 있다.

### Multi-thread

하나의 Process 안에서 여러 Thread가 일을 나누는 방식입니다.

장점:

- 같은 Heap과 자원을 쉽게 공유한다.
- Process 간 통신보다 데이터 공유가 간단하다.
- 일반적으로 Process보다 생성과 전환 비용이 작다.

단점:

- 공유 데이터 때문에 Race Condition이 발생할 수 있다.
- 잘못된 동기화로 Deadlock이 발생할 수 있다.
- 하나의 Thread가 Process 전체 상태를 망가뜨릴 수 있다.

---

## 6. Thread가 많으면 무조건 빠를까?

아닙니다.

Thread를 너무 많이 만들면 다음 비용이 생깁니다.

- Thread Stack 메모리
- Scheduling 비용
- Context Switching 비용
- Lock 경쟁
- Cache 효율 저하

CPU Core가 4개인데 CPU-bound 작업 Thread를 수백 개 만든다고 100배 빨라지지 않습니다.
오히려 CPU가 Thread를 자주 바꾸느라 비용이 커질 수 있습니다.

반면 I/O-bound 작업은 Thread가 대기하는 시간이 길기 때문에 어느 정도 동시성을 늘리는 것이 도움이 될 수 있습니다.

---

## 7. 백엔드 서버와 Thread Pool

웹 서버가 요청 하나마다 새 Thread를 만든다고 생각해봅시다.

```text
Request 1 -> create Thread
Request 2 -> create Thread
Request 3 -> create Thread
...
Request 10000 -> create Thread
```

트래픽이 갑자기 증가하면 Thread 생성 비용과 메모리 사용량이 폭증할 수 있습니다.

그래서 많은 Runtime과 서버 프레임워크는 미리 일정 수의 Thread를 만들어 재사용하는 **Thread Pool**을 사용합니다.

```text
Incoming Requests
      ↓
    Queue
      ↓
Thread Pool
├── Worker 1
├── Worker 2
├── Worker 3
└── Worker 4
```

핵심은 요청 수와 Thread 수를 무조건 1:1로 만들지 않고, 제한된 Worker를 효율적으로 재사용하는 것입니다.

---

## 8. C#/.NET과 연결

C# 백엔드에서 다음 개념은 모두 Process/Thread와 연결됩니다.

- `Thread`
- `Task`
- Thread Pool
- `async/await`
- Lock
- Parallel Processing

하지만 `Task = Thread`라고 보면 안 됩니다.

Task는 비동기 작업을 표현하는 더 높은 수준의 추상화이고, 반드시 새 Thread 하나와 1:1 대응하지 않습니다.

특히 비동기 I/O에서는 Thread가 I/O가 끝날 때까지 계속 붙잡혀 있지 않을 수 있습니다.

이 부분은 Runtime과 Async I/O 파트에서 다시 다룹니다.

---

## 9. 자주 하는 오해

### “Process가 CPU에서 직접 실행된다”

실제로 CPU가 실행하는 단위는 Thread입니다. Process는 Thread와 자원을 담는 실행 환경에 가깝습니다.

### “같은 Process의 Thread는 모든 것을 공유한다”

아닙니다. Heap과 전역 데이터는 공유할 수 있지만 Stack과 Register 상태는 Thread별로 따로 가집니다.

### “Thread가 많을수록 성능이 좋아진다”

아닙니다. Scheduling, Context Switch, Memory, Lock 경쟁 비용이 증가할 수 있습니다.

### “Task는 Thread다”

아닙니다. .NET Task는 작업을 표현하는 추상화이며 Thread와 1:1 대응하지 않습니다.

---

## 10. 60초 면접 답변

> Process는 운영체제가 독립적인 주소 공간과 자원을 가진 실행 단위로 관리하는 프로그램의 인스턴스입니다. Thread는 그 Process 안에서 실제 명령을 실행하는 흐름입니다. 같은 Process의 Thread들은 Heap, Code, 열린 파일 같은 자원을 공유하지만 각자 Stack과 Register 상태를 가집니다. Process 간에는 메모리가 기본적으로 격리되어 안정성이 높지만 통신 비용이 더 크고, Thread는 공유가 쉬운 대신 Race Condition이나 Deadlock 같은 동기화 문제가 생길 수 있습니다. 백엔드 서버에서는 요청마다 Thread를 새로 만들기보다 Thread Pool로 Worker를 재사용해 생성 비용과 자원 사용을 제어합니다.

---

## 핵심 요약

```text
Process = 독립된 주소 공간과 자원을 가진 실행 환경
Thread  = Process 안의 실제 실행 흐름

같은 Process의 Thread
공유: Heap / Code / Global Data / Files / Sockets
개별: Stack / Registers / Program Counter
```

다음 주제:

- Process Address Space
- Stack과 Heap
- Virtual Memory
