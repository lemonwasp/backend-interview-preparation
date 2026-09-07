# 11. IDisposable / Resource Lifetime

## 한 줄 요약

GC는 managed memory를 회수하지만 **파일 핸들, 소켓, DB connection, stream 같은 외부 자원의 수명까지 적절한 시점에 보장해 주지는 않는다.** 그래서 .NET에서는 `IDisposable`과 `using`으로 자원 해제를 명시적으로 관리한다.

## 파인만식 설명

GC는 방 청소부에 가깝다. 방 안의 쓰레기는 치워주지만, 빌린 렌터카를 제시간에 반납해 주지는 않는다.

파일, 소켓, DB connection 같은 자원은 OS나 외부 시스템의 제한된 자원이다. 이런 자원은 "언젠가 GC가 알아서"가 아니라 **필요가 끝난 즉시 반환**해야 한다.

```csharp
using var stream = File.OpenRead(path);
// 사용
```

`using`은 scope가 끝날 때 `Dispose()`가 호출되도록 보장한다.

## GC와 Dispose는 역할이 다르다

- GC: 도달 불가능한 managed object 메모리 회수
- Dispose: 제한된 외부 자원을 즉시 정리

`Dispose()`는 object 자체를 heap에서 없애는 기능이 아니다.

## Finalizer는 왜 최후의 수단인가

Finalizer는 정확한 실행 시점을 보장하지 않고 GC 비용도 늘릴 수 있다. 따라서 가능하면 `SafeHandle`이나 `IDisposable` 패턴을 이용해 deterministic cleanup을 우선한다.

## IAsyncDisposable

비동기 정리가 필요한 자원은 `IAsyncDisposable`과 `await using`을 쓸 수 있다.

```csharp
await using var resource = await OpenAsyncResource();
```

정리 과정 자체가 네트워크 flush 같은 비동기 I/O를 필요로 할 때 유용하다.

## Backend에서 중요한 사례

### DB Connection
Connection Pool을 쓴다고 해도 connection wrapper를 dispose하지 않으면 pool에 제때 반환되지 않을 수 있다.

### Stream
대용량 파일 처리에서 stream을 제때 닫지 않으면 file handle 고갈이나 파일 잠금 문제가 생길 수 있다.

### HttpClient
여기서는 반대로 매 요청마다 `new HttpClient()` 후 dispose하는 패턴이 connection churn을 키울 수 있다. 일반적인 서버 애플리케이션에서는 `HttpClientFactory`나 장수명 client를 사용해 handler/connection 재사용을 고려한다.

즉 **모든 IDisposable을 무조건 짧게 생성/폐기하는 것이 정답은 아니다. 자원의 소유권과 수명 모델을 이해해야 한다.**

## Ownership

가장 중요한 질문은 "누가 이 자원을 만들었고 누가 정리 책임을 가지는가?"다.

- 직접 생성했다 → 보통 직접 dispose
- DI container가 생성했다 → container가 수명 관리할 수 있음
- borrowed object를 받았다 → 호출자가 dispose하면 안 될 수 있음

## 흔한 오해

- `using`은 GC를 즉시 실행한다 → 아니다.
- `Dispose()`는 object memory를 삭제한다 → 아니다.
- GC가 있으니 resource leak이 없다 → 아니다.
- `HttpClient`도 무조건 using으로 매 요청 생성/폐기해야 한다 → 서버에서는 connection 재사용 때문에 오히려 문제가 될 수 있다.

## 60초 면접 답변

`.NET의 GC는 managed memory를 자동 회수하지만 OS handle이나 socket, stream, DB connection 같은 제한된 자원의 적시 해제까지 보장하지는 않습니다. 이런 자원은 IDisposable과 using을 통해 deterministic하게 정리합니다. Dispose는 객체 메모리를 직접 제거하는 게 아니라 내부 자원을 반환하는 것이고, 비동기 정리가 필요하면 IAsyncDisposable과 await using을 사용할 수 있습니다. 중요한 것은 ownership과 lifetime을 명확히 하는 것입니다. 예를 들어 DB connection은 사용 후 pool로 반환해야 하지만 HttpClient는 매 요청마다 생성/폐기하면 connection churn을 만들 수 있어 factory나 장수명 재사용 패턴을 씁니다.