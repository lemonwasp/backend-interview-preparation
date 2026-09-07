# 14. ExecutionContext / Context Flow

## 한 줄 요약

`.NET`의 `ExecutionContext`는 비동기 작업을 넘어갈 때 **호출 컨텍스트를 함께 흘려보내는 메커니즘**이다. 대표적으로 `AsyncLocal<T>` 기반 값, 보안/문화권 정보 등이 continuation을 따라갈 수 있다.

## 왜 필요한가

HTTP 요청을 처리하는 동안 다음 정보가 여러 async 호출을 넘어가야 할 수 있다.

- correlation / trace id
- request-scoped metadata
- tenant id
- culture

직접 모든 메서드 parameter로 전달할 수도 있지만, 일부 framework/runtime context는 ambient context로 전파된다.

## ExecutionContext와 Thread는 다르다

비동기 코드가 `await` 이후 다른 ThreadPool worker에서 계속될 수 있어도 logical context는 이어질 수 있다.

즉:

```text
Thread A -> await -> Thread B
```

이어도 logical execution context는 보존될 수 있다.

## AsyncLocal<T>

`AsyncLocal<T>`은 비동기 호출 흐름에 붙는 ambient data를 만들 때 사용할 수 있다.

```csharp
static readonly AsyncLocal<string?> CorrelationId = new();
```

하지만 global variable처럼 남용하면 의존성이 숨고 테스트가 어려워질 수 있다.

## SynchronizationContext와 차이

- `ExecutionContext`: logical context data를 전달
- `SynchronizationContext`: continuation을 어디에서 실행할지 scheduling에 관여

둘은 완전히 다른 문제를 다룬다.

ASP.NET Core는 일반적인 UI 앱과 달리 요청마다 전통적인 SynchronizationContext에 의존하지 않는다. 그래서 classic ASP.NET/UI에서 흔한 sync-over-async deadlock 패턴과는 차이가 있지만 `.Result`/`.Wait()`는 여전히 ThreadPool blocking과 scalability 문제를 만들 수 있다.

## Activity / Trace Context

현대 .NET observability에서는 `System.Diagnostics.Activity`와 OpenTelemetry가 trace/span context 전파에 중요하다.

서비스 A -> 서비스 B -> DB 호출이 하나의 trace로 연결되려면 context propagation이 필요하다.

## ExecutionContext.SuppressFlow

특수한 고성능 코드에서 context capture 비용을 피하려고 flow를 억제할 수 있지만, trace/security/ambient data가 끊길 수 있어 매우 신중해야 한다.

## Backend 위험

- tenant/user 정보가 잘못 전파되면 보안 문제
- background task가 request-scoped context를 오래 잡고 있으면 lifetime 문제
- fire-and-forget에서 context가 예상과 다르게 유지/소실될 수 있음

## 흔한 오해

- async context = 현재 Thread의 field → 아니다.
- ExecutionContext와 SynchronizationContext는 같은 것 → 아니다.
- AsyncLocal은 DI 대신 아무 데나 써도 된다 → 아니다.

## 60초 면접 답변

`ExecutionContext`는 async continuation이 다른 Thread에서 실행되더라도 logical call context를 전파하기 위한 .NET 메커니즘입니다. AsyncLocal 값이나 일부 runtime context가 이 흐름을 따라갈 수 있습니다. 반면 SynchronizationContext는 continuation을 어디에서 실행할지와 관련된 별도 개념입니다. Backend에서는 trace id나 request metadata 전파에 중요하지만 ambient state를 남용하면 의존성이 숨고, tenant/user context를 잘못 다루면 보안 문제가 생길 수 있습니다. Observability에서는 Activity와 OpenTelemetry trace context propagation과도 연결됩니다.