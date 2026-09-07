# 07. Cancellation and Timeout — Quiz

상태: **답변 대기**

1. Timeout과 Cancellation의 차이는 무엇인가?
2. `CancellationToken`은 왜 강제 Thread 종료 기능이 아닌가?
3. cooperative cancellation이란 무엇인가?
4. ASP.NET Core의 `RequestAborted`를 downstream까지 전달하는 이유는?
5. `Task.WhenAny(workTask, Task.Delay(...))`만으로 underlying 작업이 취소되지 않는 이유는?
6. cancellation이 DB COMMIT이나 외부 결제 Side Effect를 자동으로 Rollback하지 못하는 이유는?
7. CPU-bound loop에서 cancellation에 반응하려면 어떻게 해야 하는가?
8. 전체 요청 timeout이 2초인데 DB timeout이 10초이면 어떤 문제가 생길 수 있는가?
9. linked `CancellationTokenSource`는 언제 유용한가?
10. Client disconnect와 실제 서버 장애 timeout을 로그에서 구분해야 하는 이유는?
11. cancellation propagation이 안 될 때 Connection Pool과 서버 부하에 어떤 영향이 생길 수 있는가?
12. 60초 안에 Timeout / Cancellation / Deadline의 관계를 설명하라.

## 평가 기준

- Timeout policy와 Cancellation mechanism을 구분한다.
- cooperative cancellation을 설명한다.
- cancellation != rollback을 이해한다.
- request budget과 resource protection을 연결한다.

## 평가 결과

- 점수: Pending
- 보완할 개념: Pending
- 재시험: Pending
