# Quiz — async / await Internals

상태: **답변 대기**

1. C# 컴파일러는 async method를 어떤 구조로 바꾸나요?
2. await 대상 Task가 아직 끝나지 않았을 때 어떤 일이 일어나는지 순서대로 설명하세요.
3. `await`와 `.Result`의 가장 중요한 실행 차이는 무엇인가요?
4. `await`가 새 Thread를 만드는 문법이 아닌 이유는 무엇인가요?
5. async와 parallelism이 다른 이유는 무엇인가요?
6. ASP.NET Core에서 await 이후 같은 Thread가 보장되지 않는 이유는 무엇인가요?
7. SynchronizationContext가 무엇이며 UI/전통 ASP.NET과 ASP.NET Core에서 왜 의미가 다른가요?
8. sync-over-async가 ThreadPool starvation을 만드는 과정을 설명하세요.
9. CPU-bound 작업에 async를 붙인다고 계산 자체가 빨라지지 않는 이유는 무엇인가요?
10. OS의 I/O completion과 .NET continuation은 어떻게 연결되나요?
11. async method에도 allocation/관리 비용이 있을 수 있는 이유는 무엇인가요?
12. `ValueTask`를 모든 곳에 쓰면 안 되는 이유는 무엇인가요?

## 평가 기준

- state machine과 continuation을 설명한다.
- blocking과 asynchronous suspension을 구분한다.
- async/concurrency/parallelism을 구분한다.
- ASP.NET Core의 context 특성을 과도하게 단순화하지 않는다.
- OS I/O와 runtime scheduling을 연결한다.

## 평가 결과

- 점수: Pending
- 보완점: Pending
- Re-test: Pending
