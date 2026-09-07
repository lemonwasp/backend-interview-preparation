# 02. Thread Pool

## 한 줄 정의

**Thread Pool은 작업마다 새 Thread를 만들지 않고, 미리 관리되는 Worker Thread 집합을 재사용해 실행 비용과 자원 사용을 줄이는 메커니즘이다.**

---

## 왜 필요한가

Thread 생성은 공짜가 아니다.

새 Thread에는 대략 다음 비용이 따른다.

- Thread stack
- kernel/runtime metadata
- scheduler 관리 비용
- context switching 증가 가능성
- 생성/종료 비용

백엔드 서버가 요청 10,000개마다 Thread를 새로 만들면 비효율적이다.

그래서 Runtime은 Thread를 재사용한다.

```text
Incoming Work
   ↓
Work Queue
   ↓
ThreadPool Workers
   ↓
Execute Work
```

## .NET ThreadPool

.NET에서는 많은 `Task`와 async continuation이 ThreadPool 위에서 실행된다.

중요한 점:

> `Task.Run(...)`은 "새 Thread를 만든다"가 아니라 보통 ThreadPool에 작업을 등록한다.

```csharp
await Task.Run(() => CpuHeavyWork());
```

이 코드는 일반적으로 ThreadPool worker를 사용한다.

## CPU-bound와 I/O-bound

### CPU-bound

CPU 계산을 실제로 해야 한다.

예:

- 이미지 변환
- 압축
- 암호화
- 대규모 parsing

이 경우 실행 중인 동안 Thread/Core 자원이 필요하다.

### I/O-bound

DB, Network, File I/O 결과를 기다리는 시간이 크다.

올바른 async I/O에서는 기다리는 동안 worker thread를 계속 붙잡지 않을 수 있다.

```text
Bad
Thread → DB 요청 → 계속 기다림 → 결과

Better async
Thread → DB 요청 → 반환
                  ↓
            I/O completion
                  ↓
           continuation 예약
```

## ThreadPool Starvation

ThreadPool의 worker들이 오래 blocking되어 새 작업을 처리할 Thread가 부족해지는 현상이다.

예:

```csharp
var result = SomeAsyncMethod().Result;
```

또는:

- 긴 `Thread.Sleep`
- synchronous network I/O
- lock을 잡고 오래 대기
- CPU-heavy 작업 과다 제출

등이 많이 쌓이면 발생할 수 있다.

증상:

- CPU는 100%가 아닌데 요청 latency 급증
- queue 길이 증가
- timeout 증가
- worker thread 증가가 뒤늦게 따라옴

## Thread를 늘리면 항상 해결될까

아니다.

Thread가 너무 많으면:

- context switch 증가
- cache locality 악화
- stack memory 증가
- DB connection 등 downstream 병목은 그대로

즉 병목이 DB라면 worker thread를 더 만드는 것은 오히려 대기 요청만 늘릴 수 있다.

## ASP.NET Core와 연결

대략적인 흐름:

```text
Request
→ runtime schedules work
→ ThreadPool worker executes application code
→ await DB I/O
→ worker released
→ DB completion
→ continuation scheduled
→ another worker continues
→ response
```

`await` 이후 같은 Thread가 반드시 돌아온다고 가정하면 안 된다.

## TIFF→PDF 사례와 연결

페이지 30장을 무조건 `Task.Run` 30개로 병렬화한다고 항상 빨라지는 것은 아니다.

CPU-bound라면 core 수보다 훨씬 많은 worker가 경쟁하게 되어 context switching과 contention이 늘 수 있다.

그리고 PDF 문서 객체가 thread-safe하지 않다면 병렬 merge에서 lock contention도 생길 수 있다.

## 60초 면접 답변

> Thread Pool은 작업마다 OS Thread를 생성하는 비용을 피하기 위해 재사용 가능한 worker thread를 관리하는 구조입니다. .NET의 많은 Task와 continuation은 ThreadPool 위에서 실행됩니다. CPU-bound 작업은 실제 worker와 CPU core를 점유하지만 async I/O는 대기 시간 동안 worker를 반환할 수 있어 서버 처리량에 유리합니다. 반대로 `.Result`, synchronous I/O, 긴 lock 같은 blocking 작업이 많으면 ThreadPool starvation이 발생해 CPU가 여유 있어도 latency와 queue가 증가할 수 있습니다.

## 핵심 오해

- Task = Thread가 아니다.
- ThreadPool 크기를 무조건 늘리면 해결되는 것이 아니다.
- async I/O가 작업 자체를 더 빨리 만드는 것은 아니다.
- `await` 전후에 같은 Thread가 보장되는 것은 아니다.
