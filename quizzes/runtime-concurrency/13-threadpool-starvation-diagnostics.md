# Quiz 13. ThreadPool Starvation Diagnostics

상태: **답변 대기**

## Questions

1. ThreadPool starvation을 한 문장으로 설명하라.
2. CPU saturation과 starvation의 차이는?
3. `.Result` / `.Wait()`가 starvation을 만들 수 있는 이유는?
4. CPU 사용률이 낮은데 latency가 높은 상황에서 starvation을 의심할 수 있는 이유는?
5. 어떤 runtime 지표를 함께 봐야 하는가?
6. `dotnet-counters`, `dotnet-trace`, `dotnet-dump`의 역할을 대략 설명하라.
7. lock wait와 starvation은 어떻게 연결될 수 있는가?
8. DB connection pool wait와 ThreadPool 문제를 어떻게 구분할 것인가?
9. CPU-heavy 작업을 request path에서 직접 수행하면 어떤 문제가 생길 수 있는가?
10. ThreadPool minimum thread 수를 늘리는 것만으로 해결하면 안 되는 이유는?
11. bounded queue/backpressure가 starvation 완화에 어떻게 도움이 되는가?
12. 장애 상황에서 `지표 → 가설 → 검증 → 개선` 순서로 답변해 보라.

## 평가 기준

- starvation과 CPU saturation을 구분한다.
- sync-over-async의 인과관계를 설명한다.
- runtime metrics와 tracing을 연결한다.
- 단순 thread 수 증가가 근본 해결이 아님을 이해한다.