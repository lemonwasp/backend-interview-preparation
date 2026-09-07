# 07. Cancellation and Timeout

## 한 줄 요약

Timeout은 **얼마나 기다릴지에 대한 정책**, Cancellation은 **작업에게 중단을 요청하는 협력 메커니즘**이다. 둘은 자주 함께 쓰이지만 같은 개념은 아니다.

## 파인만식 설명

작업자에게 일을 맡겼다고 하자.

- Timeout: "10초 넘으면 난 더 이상 기다리지 않을 거야."
- Cancellation: "이제 그 작업을 멈춰줘."

Timeout이 발생했다고 해서 실제 작업이 자동으로 멈추는 것은 아니다.

예를 들어 HTTP 요청을 2초 후 포기했는데 downstream 작업이 계속 DB Query나 파일 작업을 수행하면 서버 자원은 계속 소비될 수 있다.

## .NET의 CancellationToken

```csharp
async Task ProcessAsync(CancellationToken cancellationToken)
{
    cancellationToken.ThrowIfCancellationRequested();
    await DoIoAsync(cancellationToken);
}
```

`CancellationToken`은 강제 Thread 종료 장치가 아니다.

작업 코드가 token을 확인하거나, token을 지원하는 API에 전달해야 실제로 취소에 반응할 수 있다.

즉 cancellation은 기본적으로 **cooperative cancellation**이다.

## CancellationTokenSource

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(3));
await ProcessAsync(cts.Token);
```

시간 제한을 token으로 표현할 수 있다.

또 여러 취소 원인을 연결할 수도 있다.

```csharp
using var linked = CancellationTokenSource.CreateLinkedTokenSource(
    requestAborted,
    shutdownToken);
```

## ASP.NET Core 요청 취소

Client가 연결을 끊으면 `HttpContext.RequestAborted`가 취소될 수 있다.

이 token을 DB/HTTP 등 downstream 작업으로 전달하면 불필요한 작업을 줄일 수 있다.

```text
Client disconnect
→ RequestAborted
→ Service
→ Repository / HttpClient
→ downstream I/O cancellation
```

## Timeout과 Cancellation의 차이

### Timeout

정책 또는 시간 budget.

예:

- HTTP Client timeout
- DB command timeout
- request deadline

### Cancellation

실행 중인 작업에게 중단 요청을 전달하는 메커니즘.

Timeout이 cancellation을 트리거할 수 있지만 두 개념은 동일하지 않다.

## Cancellation이 즉시 성공하지 않을 수 있는 이유

작업이 token을 확인하지 않거나 underlying API가 취소를 지원하지 않으면 바로 멈추지 않는다.

CPU-bound loop도 직접 확인해야 한다.

```csharp
for (...)
{
    cancellationToken.ThrowIfCancellationRequested();
    // CPU work
}
```

너무 자주 확인하면 약간의 비용이 있고, 너무 드물게 확인하면 반응성이 떨어진다.

## Cancel과 Rollback은 다르다

중요한 오해다.

Cancellation은 "앞으로의 작업을 중단"하는 것이지 이미 발생한 Side Effect를 자동으로 되돌리는 것이 아니다.

예:

```text
DB COMMIT 성공
→ cancellation 발생
```

이미 Commit된 데이터는 token 취소로 Rollback되지 않는다.

외부 결제 API가 성공한 뒤 cancellation이 와도 결제가 자동 취소되는 것은 아니다.

## Timeout layering

호출 체인이 있다고 하자.

```text
Client → API → Service A → DB
```

전체 요청 budget이 2초인데 DB timeout이 10초면 상위 요청이 끝난 후에도 DB 작업이 계속될 수 있다.

일반적인 원칙:

```text
Downstream timeout < Remaining upstream budget
```

## Task.WhenAny 기반 timeout의 함정

```csharp
var completed = await Task.WhenAny(workTask, Task.Delay(timeout));
```

이 코드는 기다리기를 그만둘 수는 있지만 `workTask` 자체를 취소하지 않는다.

따라서 underlying 작업을 취소하려면 `CancellationToken`을 실제 작업에 전달해야 한다.

## Cancellation과 예외

취소된 비동기 작업은 일반적으로 `OperationCanceledException`/`TaskCanceledException` 경로로 나타날 수 있다.

모든 cancellation을 장애 로그 Error로 찍으면 운영 로그가 오염될 수 있다.

Client disconnect처럼 정상적인 취소와 실제 timeout 장애를 구분해 관찰해야 한다.

## Backend 장애 연결

Cancellation propagation이 안 되면:

- Client는 이미 포기
- API는 downstream 요청 지속
- DB/HTTP Connection 점유
- Thread/Task/Memory 사용 지속
- 부하 증가

즉 취소 전파는 단순 UX 기능이 아니라 **resource protection**이다.

## 흔한 오해

### "CancellationToken.Cancel() 하면 Thread가 죽는다"

아니다. 협력적 취소 신호다.

### "Timeout이면 작업도 끝났다"

아니다. caller가 기다리기를 멈춘 것과 underlying work가 멈춘 것은 별개다.

### "Cancellation이면 이전 Side Effect가 Rollback된다"

아니다.

## 60초 면접 답변

> Timeout은 작업을 얼마나 기다릴지 정하는 정책이고 Cancellation은 실행 중인 작업에 중단을 요청하는 협력 메커니즘입니다. .NET에서는 CancellationToken을 API 경계에서 downstream까지 전달해 Client disconnect나 deadline을 DB/HTTP 작업까지 전파할 수 있습니다. 다만 Cancel이 Thread를 강제 종료하는 것은 아니고 작업이나 underlying API가 token에 협력해야 합니다. 또한 timeout이 발생했다고 underlying Task가 자동으로 중단되는 것도 아니며, 이미 Commit된 DB나 외부 Side Effect를 cancellation이 되돌려주지도 않습니다. 그래서 request budget, downstream timeout, cancellation propagation을 함께 설계해야 불필요한 자원 점유를 막을 수 있습니다.
