# 04. async / await 내부 동작

## 한 줄 정의

**`async/await`는 비동기 작업을 동기 코드처럼 작성하게 해주는 문법이며, C# 컴파일러는 이를 state machine과 continuation 기반 코드로 변환한다.**

---

## 파인만식 설명

다음 코드를 보자.

```csharp
var user = await GetUserAsync();
return user.Name;
```

겉보기에는 한 줄씩 멈춰서 실행되는 것처럼 보인다.

하지만 `GetUserAsync()`가 아직 끝나지 않았다면 실제로는:

```text
1. GetUserAsync 시작
2. 미완료 Task 확인
3. 현재 async method의 상태 저장
4. 호출자에게 Task 반환
5. I/O 완료
6. continuation 예약
7. 저장된 위치부터 실행 재개
```

## State Machine

컴파일러는 `async` 메서드를 내부적으로 상태를 기억할 수 있는 구조로 바꾼다.

개념적으로:

```text
State 0: await 전
State 1: 첫 await 이후
State 2: 두 번째 await 이후
Done
```

실제 generated code는 더 복잡하지만 면접에서는 이 개념이 핵심이다.

## await가 Thread를 막는가

`await` 대상 Task가 미완료라면 일반적인 async I/O에서는 현재 Thread를 계속 block하지 않는다.

이게 다음 코드와의 핵심 차이다.

```csharp
var x = GetUserAsync().Result; // blocking
```

vs

```csharp
var x = await GetUserAsync(); // asynchronous suspension
```

## SynchronizationContext

일부 UI framework에서는 await 이후 원래 context로 돌아가려는 동작이 중요하다.

ASP.NET Core에는 전통적인 ASP.NET과 같은 요청별 `SynchronizationContext`가 기본적으로 없으므로, 서버 코드에서 await 이후 반드시 같은 Thread로 돌아온다고 보면 안 된다.

## ConfigureAwait

라이브러리 코드에서 `ConfigureAwait(false)`를 볼 수 있다.

이는 continuation이 기존 context를 캡처해 복귀하려는 동작을 피하도록 하는 데 쓰일 수 있다.

다만 ASP.NET Core에서는 요청 `SynchronizationContext`가 기본적으로 없기 때문에 예전 ASP.NET/UI 환경과 의미가 다를 수 있다.

## Sync-over-Async

가장 위험한 패턴 중 하나:

```csharp
var result = SomeAsync().Result;
```

또는:

```csharp
SomeAsync().Wait();
```

문제:

- Thread blocking
- ThreadPool starvation
- 특정 context 환경에서는 deadlock 가능성

ASP.NET Core에서는 전통적인 SynchronizationContext deadlock 패턴이 덜 일반적이지만, blocking으로 인한 scalability 저하는 여전히 문제다.

## async가 CPU-bound 작업을 빠르게 만드는가

아니다.

```csharp
await HeavyCpuWorkAsync();
```

이라는 이름만으로 CPU 계산이 싸지지 않는다.

CPU-bound는 결국 CPU core 시간이 필요하다.

`Task.Run`으로 worker에 넘길 수는 있지만 총 계산량이 사라지는 것은 아니다.

## I/O Completion과 연결

비동기 socket/file I/O는 OS의 비동기/event/completion 메커니즘과 Runtime이 연결된다.

고수준에서:

```text
Application await
→ runtime issues async I/O
→ Thread 반환 가능
→ OS handles I/O
→ completion event
→ runtime schedules continuation
→ application resumes
```

## Allocation 비용

async 메서드도 공짜는 아니다.

상황에 따라:

- Task object
- state machine
- captured variables
- continuation

등의 allocation/관리 비용이 생길 수 있다.

핫패스에서는 `ValueTask` 같은 선택지가 있지만 잘못 쓰면 복잡도만 높아진다.

## 60초 면접 답변

> C#의 async/await는 비동기 작업을 동기 코드처럼 작성하도록 해주는 문법이고, 컴파일러가 메서드를 state machine으로 변환합니다. await 대상 Task가 아직 끝나지 않았다면 현재 실행 상태를 저장하고 호출자에게 Task를 반환한 뒤, 작업 완료 시 continuation을 예약해 이어서 실행합니다. 그래서 일반적인 async I/O에서는 기다리는 동안 worker thread를 점유하지 않을 수 있습니다. 반대로 `.Result`나 `.Wait()` 같은 sync-over-async는 Thread를 block해 ThreadPool starvation을 만들 수 있습니다.

## 핵심 오해

- `await` = 새 Thread 생성이 아니다.
- `async` = 병렬 실행이 아니다.
- `async` = CPU 작업 가속이 아니다.
- await 이후 같은 Thread가 보장되는 것은 아니다.
