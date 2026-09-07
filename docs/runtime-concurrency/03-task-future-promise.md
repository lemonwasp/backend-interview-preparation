# 03. Task / Future / Promise

## 한 줄 정의

**Task/Future/Promise는 '지금 끝나지 않았지만 나중에 결과가 생길 작업'을 표현하는 추상화다.**

C#에서는 대표적으로 `Task`와 `Task<T>`를 사용한다.

---

## 파인만식 설명

Thread는 실제 일을 수행하는 일꾼에 가깝고,
Task는 "이 일이 언젠가 끝난다"는 작업 표시에 가깝다.

```text
Task = 작업/결과의 약속
Thread = 코드를 실제로 실행하는 실행 자원
```

그래서 하나의 Thread가 여러 Task를 시간상 순차적으로 처리할 수 있고,
하나의 async Task도 실행 과정에서 여러 Thread를 거칠 수 있다.

## Task의 상태

개념적으로 Task는 다음 상태를 가진다.

- 아직 완료되지 않음
- 정상 완료
- 실패(Faulted)
- 취소(Canceled)

`Task<T>`는 완료되면 `T` 결과를 제공한다.

```csharp
Task<User> task = GetUserAsync(id);
User user = await task;
```

## Task가 Thread를 의미하지 않는 대표 사례

```csharp
await httpClient.GetAsync(url);
```

네트워크 응답을 기다리는 동안 Task는 미완료 상태일 수 있지만 전용 Thread가 그 Task를 위해 계속 대기할 필요는 없다.

반대로:

```csharp
await Task.Run(() => HeavyCpuWork());
```

이 작업은 실제 CPU 계산이 필요하므로 ThreadPool worker가 실행해야 한다.

즉 Task의 존재 자체로 Thread 점유 여부를 알 수 없다.

## Future / Promise와 관계

언어와 생태계마다 이름과 API는 다르지만 공통 아이디어는 비슷하다.

- C#: `Task<T>`
- Java: `Future`, `CompletableFuture`
- JavaScript: `Promise`

전부 비동기 결과를 표현하지만 세부 scheduling, cancellation, exception semantics는 다르다.

## Continuation

Task가 끝난 뒤 해야 할 다음 작업을 continuation이라고 생각할 수 있다.

```csharp
var user = await GetUserAsync();
return user.Name;
```

개념적으로:

```text
GetUserAsync 시작
→ 미완료 Task 반환
→ 현재 메서드 일시 중단
→ Task 완료
→ continuation 예약
→ user.Name 이후 코드 실행
```

## Task.WhenAll

여러 독립 I/O 작업을 동시에 시작하고 모두 기다릴 수 있다.

```csharp
var a = GetAAsync();
var b = GetBAsync();
await Task.WhenAll(a, b);
```

하지만 이것이 항상 "여러 Thread가 병렬 실행"된다는 뜻은 아니다.

I/O-bound라면 대부분 기다리는 시간의 겹침이다.

## Task.WhenAll의 주의점

무한정 많은 작업을 한 번에 시작하면:

- downstream API overload
- DB connection pool exhaustion
- memory 증가
- rate limit 초과

가 발생할 수 있다.

동시성 제한이 필요할 수 있다.

## Exception

Task 내부 예외는 Task를 Faulted 상태로 만든다.

`await`하면 해당 예외가 호출자 흐름으로 다시 전달된다.

fire-and-forget을 무분별하게 사용하면 예외 관찰과 lifecycle 관리가 어려워진다.

## Cancellation

Cancellation은 보통 강제 Thread kill이 아니다.

`.NET`의 `CancellationToken`은 협력적 취소(cooperative cancellation) 모델이다.

작업이 token을 확인하고 취소 요청에 반응해야 한다.

## 60초 면접 답변

> Task는 Thread가 아니라 비동기 작업의 완료와 결과를 표현하는 추상화입니다. I/O-bound Task는 응답을 기다리는 동안 전용 Thread를 점유하지 않을 수 있고, CPU-bound 작업은 ThreadPool worker가 실제 계산을 수행해야 합니다. `await`는 Task 완료 후 continuation을 이어 실행하도록 만들며, `Task.WhenAll`은 여러 작업의 대기 시간을 겹칠 수 있지만 반드시 여러 Thread의 병렬 실행을 의미하지는 않습니다. 또한 과도한 동시 Task 생성은 DB나 외부 API 같은 downstream 자원을 고갈시킬 수 있으므로 concurrency limit도 같이 고려해야 합니다.

## 핵심 오해

- Task = Thread가 아니다.
- `Task.WhenAll` = 무조건 병렬 Thread 실행이 아니다.
- Cancellation = 강제 Thread 종료가 아니다.
- fire-and-forget은 공짜가 아니다.
