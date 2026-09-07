# Quiz 12. Async Stream / Channel

상태: **답변 대기**

## Questions

1. `IAsyncEnumerable<T>`는 어떤 문제를 해결하는가?
2. async stream과 일반 `IEnumerable<T>`의 차이를 설명하라.
3. async stream이 자동 병렬 처리를 의미하지 않는 이유는?
4. `Channel<T>`의 핵심 역할은 무엇인가?
5. bounded channel이 backpressure에 유리한 이유는?
6. unbounded queue가 overload 상황에서 위험한 이유는?
7. Channel과 Kafka 같은 durable broker의 차이는?
8. Producer가 Consumer보다 빠를 때 어떤 정책을 선택할 수 있는가?
9. CancellationToken을 stream/channel pipeline에 전파해야 하는 이유는?
10. writer completion 처리가 없으면 어떤 문제가 생길 수 있는가?
11. image/file pipeline에서 Channel을 사용할 때 concurrency를 무작정 늘리면 안 되는 이유는?
12. streaming이 peak memory와 time-to-first-item에 어떤 영향을 주는가?

## 평가 기준

- streaming과 parallelism을 구분한다.
- Channel을 in-process coordination으로 설명한다.
- bounded capacity/backpressure를 이해한다.
- cancellation/completion/error propagation을 연결한다.