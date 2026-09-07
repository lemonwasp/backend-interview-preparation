# 08. Concurrency vs Parallelism

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 자신의 말로 설명할 수 있어야 합니다.

- Concurrency와 Parallelism은 무엇이 다른가?
- Single Core에서도 Concurrency가 가능한 이유는 무엇인가?
- Multi Core가 있어도 Parallelism이 자동으로 성능 향상을 보장하지 않는 이유는 무엇인가?
- CPU-bound와 I/O-bound에서 동시성 전략이 왜 달라지는가?
- Thread, Task, async/await는 각각 Concurrency와 어떤 관계가 있는가?
- Shared State가 왜 동시성 문제의 핵심이 되는가?

---

## 1. 두 요리 주문을 처리한다고 생각해보자

한 명의 요리사가 두 주문을 번갈아 처리할 수 있습니다.

```text
A 썰기 → B 굽기 준비 → A 볶기 → B 굽기
```

한 순간에는 하나만 하지만 두 작업을 **겹쳐 진행**합니다.
이것이 Concurrency의 핵심입니다.

반대로 요리사 두 명이 각각 A와 B를 같은 순간에 실제로 조리하면 Parallelism입니다.

```text
Cook 1: A ─────────>
Cook 2: B ─────────>
```

한 문장으로 정리하면:

> Concurrency는 여러 작업이 같은 기간에 진행되는 구조이고,
> Parallelism은 여러 작업이 같은 순간에 실제로 실행되는 것이다.

---

## 2. Single Core에서도 Concurrency는 가능하다

CPU Core 하나는 한 순간에 하나의 실행 흐름만 처리합니다.
그런데 운영체제가 매우 빠르게 작업을 바꿔 실행하면 다음처럼 보입니다.

```text
시간 →
A A B B A C B C A ...
```

A, B, C 모두 일정 기간 동안 진행됩니다.
하지만 어느 한 순간에는 하나만 CPU를 사용합니다.

따라서:

```text
Single Core
Concurrency: 가능
Parallelism: CPU 실행 기준으로는 불가능
```

---

## 3. Multi Core에서는 Parallelism이 가능하다

4 Core CPU에서는 최대 여러 실행 흐름이 실제로 같은 순간에 계산할 수 있습니다.

```text
Core 1 → Task A
Core 2 → Task B
Core 3 → Task C
Core 4 → Task D
```

CPU-bound 작업에서는 이런 Parallelism이 처리 시간을 줄이는 데 도움이 될 수 있습니다.

하지만 작업을 100개 Thread로 나눈다고 100배 빨라지지는 않습니다.

이유:

- CPU Core 수 제한
- Scheduling 비용
- Context Switching
- Cache 경쟁
- Lock 경쟁
- 작업 분할/합치기 비용
- 순차 실행이 필요한 부분

---

## 4. Concurrency는 구조, Parallelism은 실행 상태에 가깝다

Concurrency는 프로그램이 여러 작업을 독립적으로 진행할 수 있도록 구성되어
있는지를 나타내는 개념입니다.

Parallelism은 실제 하드웨어가 여러 작업을 동시에 실행하는 상황입니다.

예를 들어 async HTTP 서버는 한 Thread에서도 여러 요청의 I/O 대기를 겹쳐 관리할
수 있습니다. 이것은 강한 Concurrency를 가지지만 모든 요청이 동시에 CPU에서
실행되는 것은 아닙니다.

---

## 5. CPU-bound에서는 Parallelism을 생각한다

CPU-bound 작업은 실제 계산 시간이 병목입니다.

예:

- 이미지 변환
- 영상 인코딩
- 압축
- 암호화
- 수치 계산

이 경우 여러 Core에 계산을 적절히 분배하면 성능 향상을 얻을 수 있습니다.

```text
4 independent chunks
↓
4 cores
↓
parallel execution
```

그러나 작업 간 의존성이 강하거나 공유 Lock이 많으면 이점이 줄어듭니다.

---

## 6. I/O-bound에서는 Concurrency가 중요하다

I/O-bound 작업은 실제 CPU 계산보다 기다리는 시간이 큽니다.

```text
Request A → DB wait ──────>
Request B → API wait ─────>
Request C → file wait ────>
```

이때 하나를 기다리는 동안 다른 요청을 진행할 수 있으면 전체 처리량이 좋아집니다.

따라서 I/O-bound 백엔드에서는 async I/O, Event Loop, 적절한 Thread Pool 같은
Concurrency 구조가 중요합니다.

---

## 7. Thread와 Concurrency

여러 Thread를 만들면 여러 실행 흐름을 구성할 수 있습니다.

```text
Process
├── Thread A
├── Thread B
└── Thread C
```

Multi Core에서는 이 Thread들이 실제 Parallel하게 실행될 수도 있습니다.

하지만 Thread 수가 곧 Parallelism 수준은 아닙니다.

```text
100 Threads on 4 Cores
≠ 100-way CPU Parallelism
```

많은 Thread는 오히려 Scheduling과 Context Switch 비용을 증가시킬 수 있습니다.

---

## 8. Task와 async/await

C#의 `Task`는 비동기 작업을 표현하는 추상화입니다.

`Task`가 있다고 해서 항상 별도 Thread가 있다는 뜻은 아닙니다.

### I/O Task

```csharp
await httpClient.GetAsync(url);
```

I/O 완료를 기다리는 동안 전용 Thread가 계속 Blocking되지 않을 수 있습니다.

### CPU Task

```csharp
await Task.Run(() => HeavyCalculation());
```

이 경우 Thread Pool Thread에서 CPU 작업을 실행할 수 있습니다.

따라서 `Task`는 작업의 표현이고, 실제 실행 방식은 CPU 작업인지 I/O 작업인지에
따라 다를 수 있습니다.

---

## 9. Shared State가 문제를 만든다

Concurrency 자체보다 더 어려운 것은 여러 실행 흐름이 **같은 변경 가능한 데이터**를
동시에 다루는 상황입니다.

```text
Thread A ─┐
          ├─> shared counter
Thread B ─┘
```

두 Thread가 동시에 `counter++`를 실행하면 기대한 결과와 다른 값이 나올 수 있습니다.

왜냐하면 `counter++`는 개념적으로 하나의 완전한 동작처럼 보여도 내부적으로는
다음처럼 여러 단계일 수 있기 때문입니다.

```text
read counter
add 1
write counter
```

이 과정이 서로 끼어들면 Race Condition이 발생합니다.

다음 문서에서 Lock과 Deadlock까지 연결합니다.

---

## 10. Amdahl's Law 직관

프로그램의 일부가 반드시 순차적으로 실행되어야 한다면 병렬화 효과에는 한계가
있습니다.

예를 들어 전체 작업의 50%만 병렬화할 수 있다면 CPU Core를 무한히 늘려도 전체
실행 시간이 무한히 줄어들지 않습니다.

정확한 공식 자체보다 다음 직관이 중요합니다.

> 병렬화 가능한 부분만 빨라지고, 순차 구간은 그대로 남는다.

따라서 성능 튜닝에서는 “Thread를 몇 개 만들까?”보다 먼저 실제 병목과 병렬화
가능 구간을 측정해야 합니다.

---

## 11. TIFF-to-PDF 사례와 연결

각 페이지 변환이 서로 독립적인 CPU-bound 작업이라고 가정하면 일부 Parallelism을
활용할 수 있습니다.

```text
Page 1 → Core 1
Page 2 → Core 2
Page 3 → Core 3
Page 4 → Core 4
```

하지만 PDF 문서 객체가 Thread-safe하지 않거나 최종 추가 순서를 보장해야 한다면
모든 단계를 무제한 병렬화할 수 없습니다.

또한 이미지 Decode/Encode가 CPU를 포화시키는 상태에서 Thread를 더 늘리면
Context Switch만 증가할 수 있습니다.

따라서:

1. 병목을 측정한다.
2. 서로 독립적인 구간을 찾는다.
3. Core 수를 고려해 병렬화 수준을 제한한다.
4. Shared State와 Thread Safety를 확인한다.
5. 전후 성능을 다시 측정한다.

이 순서가 중요합니다.

---

## 12. 자주 하는 오해

### “Concurrency = Parallelism”

아닙니다. Single Core에서도 작업을 번갈아 진행하는 Concurrency는 가능합니다.

### “Thread가 많을수록 Parallelism이 커진다”

실제 CPU Parallelism은 Core와 실행 자원에 제한됩니다.

### “async는 Parallelism이다”

아닙니다. async는 I/O 대기를 효율적으로 겹치는 Concurrency를 만드는 데 주로
유용합니다.

### “병렬화하면 항상 빨라진다”

작업 분할, 동기화, Scheduling, Cache, 순차 구간 비용 때문에 오히려 느려질 수도
있습니다.

---

## 13. 60초 면접 답변

> Concurrency는 여러 작업이 같은 기간에 겹쳐 진행될 수 있는 구조이고,
> Parallelism은 여러 작업이 실제로 같은 순간에 실행되는 것입니다. 그래서
> Single Core에서도 Context Switching이나 비동기 I/O를 통해 Concurrency는
> 가능하지만 CPU Parallelism은 여러 Core가 필요합니다. 백엔드에서는 I/O-bound
> 작업에 async 기반 Concurrency가 유리하고, CPU-bound 작업은 여러 Core를 활용한
> 적절한 Parallelism이 도움이 될 수 있습니다. 다만 Thread 수를 과도하게 늘리면
> Context Switch와 Lock 경쟁이 증가할 수 있고, Shared State가 있으면 Race
> Condition도 고려해야 합니다.

---

## 핵심 요약

```text
Concurrency = 같은 기간에 여러 작업이 진행
Parallelism = 같은 순간에 여러 작업이 실제 실행
Single Core = concurrency 가능, CPU parallelism 제한
async        = 주로 I/O concurrency
shared state = race condition의 출발점
```

다음 주제:

- Race Condition
- Lock
- Deadlock
