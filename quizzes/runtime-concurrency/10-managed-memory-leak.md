# 10. Managed Memory Leak — Quiz

상태: **답변 대기**

1. GC가 있어도 Managed Memory Leak이 발생할 수 있는 이유는?
2. GC Root와 Reachability가 leak 진단에서 왜 중요한가?
3. Static Collection이 leak을 만드는 과정을 설명하라.
4. Event Handler unsubscribe 누락이 객체 lifetime을 늘리는 이유는?
5. Unbounded Cache와 Unbounded Queue는 각각 어떤 방식으로 memory를 증가시키는가?
6. Managed Memory Leak과 Resource Leak의 차이는 무엇인가?
7. High Allocation Rate와 Memory Retention을 지표상 어떻게 구분할 수 있는가?
8. Process RSS가 높다고 Managed Heap Leak이라고 단정하면 안 되는 이유는?
9. Heap Dump를 한 번만 보는 것보다 시간차 비교가 유용한 이유는?
10. Retention Path란 무엇인가?
11. `GC.Collect()`가 근본적인 Leak 해결책이 아닌 이유는?
12. ASP.NET에서 Request 범위 객체를 static/background task에 오래 보관하면 어떤 문제가 생길 수 있는가?
13. `Dispose()`와 GC의 역할 차이를 설명하라.
14. 60초 안에 Managed Memory Leak의 원인과 진단 절차를 설명하라.

## 평가 기준

- GC가 reachable object를 수집하지 않는다는 점을 이해한다.
- leak, high allocation, native/resource leak을 구분한다.
- heap trend → dump comparison → retention path 순서로 진단한다.
- cache/queue lifetime policy와 backpressure를 연결한다.

## 평가 결과

- 점수: Pending
- 보완할 개념: Pending
- 재시험: Pending
