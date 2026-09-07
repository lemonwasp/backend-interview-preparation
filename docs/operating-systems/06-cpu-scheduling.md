# 06. CPU Scheduling

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 자신의 말로 설명할 수 있어야 합니다.

- CPU Scheduling이 왜 필요한가?
- Ready / Running / Waiting 상태는 무엇인가?
- Preemptive와 Non-preemptive Scheduling의 차이는 무엇인가?
- FCFS, SJF/SRTF, Round Robin, Priority Scheduling의 핵심 차이는 무엇인가?
- Throughput, Turnaround Time, Waiting Time, Response Time은 어떻게 다른가?
- 백엔드 서버의 Thread 수와 Scheduling 비용은 어떤 관계가 있는가?

---

## 1. CPU Scheduling은 왜 필요한가?

CPU Core 하나는 한 순간에 하나의 실행 흐름만 실제로 실행합니다.
하지만 시스템에는 동시에 실행하고 싶은 Process와 Thread가 많습니다.

```text
Thread A ─┐
Thread B ─┼─> Ready Queue ─> Scheduler ─> CPU
Thread C ─┘
```

운영체제는 **누가 다음에 CPU를 사용할지** 결정해야 합니다.
이 결정을 담당하는 것이 CPU Scheduler입니다.

한 문장으로 정리하면:

> CPU Scheduling은 제한된 CPU 시간을 여러 실행 흐름에 배분해 응답성과 처리량,
> 공정성을 조절하는 운영체제의 정책이다.

---

## 2. Thread 상태와 Scheduling

개념적으로 다음 상태를 구분하면 쉽습니다.

```text
Ready   : 실행할 준비가 되었지만 CPU를 기다리는 상태
Running : 현재 CPU에서 실행 중인 상태
Waiting : I/O나 Lock 같은 사건을 기다리는 상태
```

예를 들어 HTTP 요청을 처리하는 Thread가 DB 응답을 기다리면 CPU를 계속 붙잡을
이유가 없습니다. 해당 Thread는 Waiting 상태로 가고, Scheduler는 Ready 상태의
다른 Thread를 실행할 수 있습니다.

---

## 3. Preemptive vs Non-preemptive

### Non-preemptive

한 작업이 CPU를 잡으면 자발적으로 양보하거나 Blocking/종료될 때까지 계속
실행하게 두는 방식입니다.

장점:

- 구현이 비교적 단순하다.
- 전환 횟수가 줄 수 있다.

단점:

- 긴 작업 하나가 CPU를 오래 점유하면 다른 작업의 응답성이 나빠질 수 있다.

### Preemptive

운영체제가 실행 중인 작업을 중간에 멈추고 다른 작업에 CPU를 줄 수 있습니다.

장점:

- 인터랙티브 작업의 응답성을 높일 수 있다.
- 한 작업이 CPU를 독점하는 것을 막기 쉽다.

단점:

- Context Switch 비용이 발생한다.
- Scheduling 정책이 더 복잡해진다.

현대 범용 운영체제는 일반적으로 Preemptive Scheduling을 사용합니다.

---

## 4. 주요 Scheduling 알고리즘

### FCFS — First Come, First Served

먼저 온 작업부터 처리합니다.

```text
A(10ms), B(1ms), C(1ms)

A ────────── B C
```

구현은 단순하지만 긴 작업이 앞에 오면 뒤의 짧은 작업들이 오래 기다립니다.
이를 Convoy Effect라고 부릅니다.

### SJF — Shortest Job First

실행 시간이 가장 짧은 작업을 먼저 처리합니다.

평균 Waiting Time을 줄이는 데 유리하지만 실제 시스템에서는 작업이 앞으로
얼마나 오래 실행될지 정확히 알기 어렵습니다.

### SRTF — Shortest Remaining Time First

SJF의 Preemptive 형태입니다. 남은 실행 시간이 더 짧은 작업이 들어오면 현재
작업을 중단하고 새 작업을 실행할 수 있습니다.

### Round Robin

각 작업에 일정한 Time Quantum을 주고 순환합니다.

```text
A → B → C → A → B → ...
```

Quantum이 너무 크면 FCFS처럼 되고, 너무 작으면 Context Switch가 너무 많이
발생할 수 있습니다.

### Priority Scheduling

우선순위가 높은 작업을 먼저 실행합니다.

문제는 낮은 우선순위 작업이 계속 밀려 실행되지 못하는 Starvation입니다.
이를 완화하기 위해 오래 기다린 작업의 우선순위를 점차 올리는 Aging 같은
기법을 사용할 수 있습니다.

---

## 5. Scheduling 성능 지표

### Throughput

단위 시간 동안 완료한 작업 수입니다.

```text
100 requests / second
```

### Turnaround Time

작업이 들어온 시점부터 완전히 끝날 때까지의 시간입니다.

```text
Turnaround = Completion Time - Arrival Time
```

### Waiting Time

Ready Queue에서 CPU를 기다린 총 시간입니다.

### Response Time

요청이 들어온 뒤 **처음 반응을 시작하기까지** 걸린 시간입니다.

인터랙티브 서비스에서는 평균 완료 시간뿐 아니라 Response Time이 중요합니다.

---

## 6. CPU-bound와 I/O-bound

### CPU-bound

대부분의 시간을 계산에 씁니다.

예:

- 이미지 인코딩
- 압축
- 암호화
- 대규모 정렬

CPU Core 수보다 지나치게 많은 CPU-bound Thread를 만들면 계산량 자체는 줄지
않고 Scheduling과 Context Switch 비용만 늘 수 있습니다.

### I/O-bound

대부분의 시간을 외부 작업을 기다리는 데 씁니다.

예:

- DB 응답
- HTTP API
- 디스크 읽기
- Socket I/O

이 경우 어떤 Thread가 Waiting으로 빠지는 동안 다른 작업을 실행할 수 있어
동시성을 활용하기 좋습니다.

---

## 7. 백엔드 Thread Pool과 Scheduling

Thread Pool이 필요한 이유 중 하나는 Thread를 무제한 생성하지 않기 위해서입니다.

```text
1000 requests
    ↓
bounded thread pool
    ↓
limited runnable threads
    ↓
CPU Scheduler
```

요청마다 새 Thread를 만들면 다음 비용이 증가할 수 있습니다.

- Stack 메모리
- Thread 생성/정리 비용
- Ready Thread 수
- Context Switching
- CPU Cache/TLB 손실

따라서 중요한 것은 “Thread가 많을수록 빠르다”가 아니라 워크로드와 CPU/I/O
특성에 맞는 동시성 수준을 선택하는 것입니다.

---

## 8. TIFF-to-PDF 사례와 연결

이미지 변환 작업이 CPU-bound라면 페이지마다 Thread를 무제한으로 만드는 것은
좋은 전략이 아닐 수 있습니다.

```text
30 pages
↓
30 CPU-bound workers
↓
4-core CPU
↓
많은 Ready Thread + Scheduling + Context Switching
```

4 Core 환경이라면 동시에 실제 계산을 수행할 수 있는 CPU 실행 흐름은 제한적입니다.
따라서 병렬화 수준을 측정 없이 크게 올리면 오히려 성능이 나빠질 수도 있습니다.

반대로 파일이나 네트워크 대기가 큰 작업이라면 적절한 비동기 I/O나 제한된
동시성으로 CPU 유휴 시간을 줄일 수 있습니다.

---

## 9. 자주 하는 오해

### “Round Robin이 항상 가장 공정하고 가장 빠르다”

공정성과 응답성에는 유리할 수 있지만 Quantum과 워크로드에 따라 Context Switch
비용이 커질 수 있습니다.

### “CPU 사용률 100%면 무조건 좋은 상태다”

유용한 계산으로 100%인지, 과도한 Thread 경쟁과 Scheduling으로 소모되는지
구분해야 합니다.

### “Thread를 많이 만들면 CPU Core를 더 많이 쓸 수 있다”

Core 수 자체가 늘어나는 것은 아닙니다. 과도한 Runnable Thread는 Context Switch와
Cache 손실을 늘릴 수 있습니다.

---

## 10. 60초 면접 답변

> CPU Scheduling은 여러 Process나 Thread가 제한된 CPU 시간을 공유하도록
> 운영체제가 다음 실행 대상을 선택하는 과정입니다. 대표적인 정책으로 FCFS,
> SJF, Round Robin, Priority Scheduling 등이 있고, 각각 처리량, 응답시간,
> 공정성 사이의 trade-off가 있습니다. 현대 운영체제는 일반적으로 Preemptive
> Scheduling을 사용해 한 작업이 CPU를 독점하지 못하게 합니다. 백엔드에서는
> Thread 수를 무작정 늘리면 Ready Queue와 Context Switch가 증가할 수 있기
> 때문에 CPU-bound인지 I/O-bound인지에 따라 동시성 수준을 조절해야 합니다.

---

## 핵심 요약

```text
Scheduler = 다음 CPU 실행 대상을 선택
Ready     = CPU를 기다리는 상태
Waiting   = I/O/Lock 등 사건을 기다리는 상태
많은 Thread ≠ 항상 높은 성능
```

다음 주제:

- Blocking / Non-blocking
- Synchronous / Asynchronous
