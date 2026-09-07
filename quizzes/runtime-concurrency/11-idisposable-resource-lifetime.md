# Quiz 11. IDisposable / Resource Lifetime

상태: **답변 대기**

## Questions

1. GC가 있는데도 `IDisposable`이 필요한 이유는 무엇인가?
2. `Dispose()`와 GC의 역할 차이를 설명하라.
3. `using`이 내부적으로 보장하려는 것은 무엇인가?
4. Finalizer에만 의존하면 안 되는 이유는?
5. `IAsyncDisposable`은 언제 필요한가?
6. DB Connection을 dispose하는 것이 왜 중요하며 Connection Pool과 어떤 관계인가?
7. `HttpClient`를 매 요청마다 생성하고 dispose하는 패턴이 왜 문제가 될 수 있는가?
8. Resource ownership이 중요한 이유는 무엇인가?
9. DI Container가 만든 disposable object를 임의로 dispose하면 어떤 문제가 생길 수 있는가?
10. Managed memory leak과 unmanaged/resource leak의 차이를 설명하라.
11. `Dispose()`를 호출했다고 해서 object가 즉시 heap에서 사라지는가?
12. Backend에서 file handle 고갈이 발생했다면 어떤 lifetime 문제를 의심할 수 있는가?

## 평가 기준

- GC와 deterministic cleanup을 구분한다.
- `using` / `await using`의 목적을 설명한다.
- Connection Pool과 Dispose 관계를 설명한다.
- HttpClient lifetime의 예외적 특성을 안다.
- ownership/lifetime을 코드 구조와 연결한다.