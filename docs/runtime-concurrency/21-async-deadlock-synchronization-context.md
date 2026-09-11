# Async Deadlocks, SynchronizationContext, and ConfigureAwait

## 한 줄 정의

**비동기 코드도 잘못된 동기 대기와 context 복귀 규칙이 결합되면 deadlock이나 thread starvation을 만들 수 있으며, 특히 `.Result`, `.Wait()`, `SynchronizationContext`, `ConfigureAwait`의 관계를 이해해야 한다.**

---

## 1. async면 deadlock이 사라지는가?

아니다.

`async/await`는 Thread를 효율적으로 사용하도록 돕지만 다음과 같은 코드가 문제를 만들 수 있다.

```csharp
var result = GetDataAsync().Result;
```

또는:

```csharp
GetDataAsync().Wait();
```

비동기 작업을 동기적으로 기다리는 것을 흔히 **sync-over-async**라고 한다.

---

## 2. 고전적인 deadlock 시나리오

UI/legacy ASP.NET처럼 특정 `SynchronizationContext`가 있는 환경을 생각하자.

```csharp
async Task<string> GetDataAsync()
{
    await httpClient.GetStringAsync(url);
    return "done";
}

string result = GetDataAsync().Result;
```

흐름은 다음과 같다.

```text
1. 현재 Thread가 GetDataAsync 실행
2. await에서 비동기 I/O 시작
3. caller가 .Result로 현재 Thread를 block
4. I/O 완료
5. continuation이 원래 context로 돌아오려 함
6. 원래 Thread는 .Result에서 막혀 있음
7. 서로 기다림
```

결과적으로 deadlock이 발생할 수 있다.

---

## 3. SynchronizationContext란?

간단히 말하면 continuation을 **어디에서 실행할지** 조정하는 abstraction이다.

예를 들어 UI에서는 UI control을 특정 Thread에서만 접근해야 한다.

```text
await 이전: UI Thread
await 이후: UI Thread로 복귀
```

이 동작은 편리하지만 sync-over-async와 만나면 문제가 될 수 있다.

---

## 4. ConfigureAwait(false)

라이브러리 코드에서는 다음 형태를 볼 수 있다.

```csharp
await SomeIoAsync().ConfigureAwait(false);
```

의미는 대략:

> 현재 context로 반드시 복귀할 필요가 없다.

따라서 continuation이 특정 context를 기다리는 문제를 줄일 수 있다.

하지만 중요한 점:

```text
ConfigureAwait(false)는 모든 async 문제를 해결하는 마법이 아니다.
```

caller가 `.Result`를 남발하거나 다른 resource cycle이 있으면 여전히 문제가 생길 수 있다.

---

## 5. ASP.NET Core에서는?

ASP.NET Core는 전통적인 ASP.NET과 달리 일반적으로 요청마다 classic `SynchronizationContext`를 사용하지 않는다.

따라서 고전적인 "context capture + .Result" deadlock 패턴은 덜 직접적이다.

하지만 `.Result`와 `.Wait()`는 여전히 위험할 수 있다.

왜냐하면 Thread Pool worker를 block하기 때문이다.

```text
request
 -> worker thread
 -> async operation을 .Result로 대기
 -> worker가 아무 일도 못하고 점유됨
```

트래픽이 많으면 Thread Pool starvation으로 이어질 수 있다.

---

## 6. Deadlock과 Starvation 구분

### Deadlock

작업들이 서로가 가진 조건을 기다려 영원히 progress하지 못한다.

```text
A waits B
B waits A
```

### Thread Pool Starvation

작업이 원칙적으로 완료 가능하지만 실행할 worker가 부족해 매우 늦어진다.

```text
많은 worker가 blocking
        ↓
continuation 실행할 worker 부족
        ↓
latency 폭증
```

증상은 비슷해 보여도 원인이 다르다.

---

## 7. async all the way

가장 중요한 원칙 중 하나다.

```text
Controller async
 -> Service async
 -> Repository async
 -> I/O async
```

중간에서:

```csharp
.Result
.Wait()
```

로 다시 동기 흐름으로 바꾸지 않는 것이 좋다.

---

## 8. Cancellation도 함께 전달한다

비동기 작업은 무한히 기다리지 않도록 cancellation과 timeout도 설계해야 한다.

```csharp
async Task<Data> LoadAsync(
    CancellationToken cancellationToken)
{
    return await repository.LoadAsync(cancellationToken);
}
```

상위 request가 취소됐는데 하위 작업이 계속 실행되면 불필요한 자원을 소비한다.

---

## 9. CPU-bound 작업과 async

`async`는 CPU 작업을 자동으로 병렬화하지 않는다.

```csharp
async Task<int> CalculateAsync()
{
    return ExpensiveCpuCalculation();
}
```

위 코드에 `async`를 붙였다고 CPU 작업이 사라지는 것이 아니다.

I/O-bound와 CPU-bound를 구분해야 한다.

```text
I/O-bound -> async/await가 매우 유용
CPU-bound -> parallelism / worker scheduling 별도 고려
```

---

## 10. 예외 처리

비동기 예외는 `await`할 때 다시 관찰된다.

```csharp
try
{
    await DoWorkAsync();
}
catch (Exception ex)
{
    // handle
}
```

`async void`는 일반 이벤트 핸들러 외에는 피하는 편이 좋다.

호출자가 Task를 받아 완료/실패를 추적할 수 없기 때문이다.

---

## 11. 면접 질문

### Q. `.Result`가 왜 위험한가요?

> 특정 SynchronizationContext에서는 continuation이 원래 context로 돌아오려 하는데 그 Thread가 `.Result`로 막혀 deadlock이 생길 수 있습니다. ASP.NET Core에서도 classic context deadlock 가능성은 낮지만 worker thread를 block해 Thread Pool starvation을 만들 수 있습니다.

### Q. ConfigureAwait(false)는 언제 쓰나요?

> 특정 caller context로 돌아갈 필요가 없는 library code에서 context capture를 피할 때 사용할 수 있습니다. 다만 application code에서 무조건 붙이는 규칙이라기보다 실행 context 요구사항을 이해하고 사용해야 합니다.

### Q. async all the way가 무슨 뜻인가요?

> 비동기 call chain 중간에서 `.Wait()`나 `.Result`로 동기 대기를 만들지 않고 상위 레이어까지 Task를 전파해 끝까지 await하는 원칙입니다.

---

## 12. 60초 답변

> async 코드도 deadlock이 발생할 수 있습니다. 대표적으로 특정 SynchronizationContext가 있는 환경에서 비동기 메서드가 await 후 원래 context로 돌아오려는데 caller가 `.Result`나 `.Wait()`로 그 Thread를 막고 있으면 서로 기다릴 수 있습니다. ASP.NET Core에서는 classic SynchronizationContext deadlock은 덜하지만 sync-over-async가 worker thread를 점유해 Thread Pool starvation을 만들 수 있습니다. 그래서 가능한 한 async all the way를 유지하고, 필요하면 library code에서 `ConfigureAwait(false)`를 사용하며 cancellation과 timeout도 함께 전달합니다.

## 기억할 핵심

```text
1. async != deadlock 불가능.
2. .Result / .Wait()는 sync-over-async를 만든다.
3. SynchronizationContext는 continuation 실행 위치와 관련된다.
4. ASP.NET Core에서도 blocking은 Thread Pool starvation을 만들 수 있다.
5. 기본 원칙은 async all the way다.
```
