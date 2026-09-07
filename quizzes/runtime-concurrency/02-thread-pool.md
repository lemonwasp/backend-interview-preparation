# Quiz — Thread Pool

상태: **답변 대기**

1. Thread Pool이 필요한 이유를 Thread 생성 비용과 연결해 설명하세요.
2. `Task.Run()`이 새 Thread를 만든다고 보면 왜 부정확한가요?
3. CPU-bound와 I/O-bound 작업의 차이는 무엇인가요?
4. async I/O가 worker thread 사용 효율을 높이는 이유는 무엇인가요?
5. ThreadPool starvation이 무엇인가요?
6. `.Result` / `.Wait()`가 서버에서 위험할 수 있는 이유는 무엇인가요?
7. CPU 사용률이 낮은데 latency가 급증했다면 ThreadPool starvation을 왜 의심할 수 있나요?
8. ThreadPool worker 수를 무작정 늘리면 생길 수 있는 비용은 무엇인가요?
9. `await` 이후 같은 Thread가 실행을 이어간다고 보장할 수 있나요?
10. CPU-bound TIFF 페이지 변환을 페이지 수만큼 `Task.Run`하는 전략의 문제를 설명하세요.
11. ThreadPool과 DB Connection Pool은 어떻게 다른 종류의 자원 풀인가요?

## 평가 기준

- Task와 Thread를 구분한다.
- CPU-bound / I/O-bound 차이를 causal하게 설명한다.
- starvation의 원인과 증상을 연결한다.
- Thread 수 증가의 trade-off를 말한다.
- 실제 backend request 흐름과 연결한다.

## 평가 결과

- 점수: Pending
- 보완점: Pending
- Re-test: Pending
