# Quiz 15. Runtime Observability

상태: **답변 대기**

## Questions

1. Metrics / Logs / Traces의 역할 차이는?
2. .NET Runtime에서 자주 보는 핵심 지표를 5개 이상 말하라.
3. CPU는 낮은데 latency가 높다면 어떤 가설을 세울 수 있는가?
4. ThreadPool queue length와 thread count를 왜 같이 봐야 하는가?
5. Managed heap과 RSS의 차이를 진단에서 어떻게 활용하는가?
6. Gen 2 GC와 LOH가 tail latency에 영향을 줄 수 있는 이유는?
7. `dotnet-counters`, `dotnet-trace`, `dotnet-dump`, `dotnet-gcdump`를 언제 쓰는가?
8. distributed trace가 slow downstream을 찾는 데 어떻게 도움이 되는가?
9. 평균 latency만 보면 안 되는 이유는?
10. RED와 USE는 각각 무엇을 관찰하는 프레임인가?
11. allocation 최적화를 측정 없이 먼저 하면 안 되는 이유는?
12. `지표 → 가설 → trace/profile → 수정 → 재측정` 흐름으로 장애 진단 답변을 구성하라.

## 평가 기준

- metrics/logs/traces를 구분한다.
- ThreadPool/GC/memory/lock 지표를 실제 장애와 연결한다.
- managed heap과 process RSS를 구분한다.
- profile-before-optimize 원칙을 이해한다.