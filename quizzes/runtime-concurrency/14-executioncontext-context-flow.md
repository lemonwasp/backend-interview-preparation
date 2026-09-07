# Quiz 14. ExecutionContext / Context Flow

상태: **답변 대기**

## Questions

1. ExecutionContext는 무엇을 위해 존재하는가?
2. async continuation이 다른 Thread에서 실행되어도 logical context가 이어질 수 있는 이유는?
3. `AsyncLocal<T>`은 어떤 용도로 쓸 수 있는가?
4. ExecutionContext와 SynchronizationContext의 차이는?
5. ASP.NET Core와 classic ASP.NET/UI의 SynchronizationContext 차이는 왜 중요한가?
6. `.Result` / `.Wait()`가 ASP.NET Core에서도 문제가 되는 이유는?
7. trace/correlation id 전파와 ExecutionContext는 어떻게 연결되는가?
8. `Activity`와 OpenTelemetry는 어떤 역할을 하는가?
9. `ExecutionContext.SuppressFlow()`를 함부로 쓰면 안 되는 이유는?
10. ambient context를 남용하면 테스트/설계에 어떤 문제가 생기는가?
11. tenant/user context 전파가 보안 문제로 이어질 수 있는 시나리오는?
12. fire-and-forget task에서 context/lifetime 문제를 설명하라.

## 평가 기준

- logical context와 Thread-local state를 구분한다.
- ExecutionContext와 SynchronizationContext를 구분한다.
- ASP.NET Core의 특성을 설명한다.
- tracing과 context propagation을 연결한다.