# 06. Concurrent Collections — Quiz

상태: **답변 대기**

1. `Dictionary`와 `ConcurrentDictionary`의 핵심 차이는 무엇인가?
2. `ContainsKey` 후 `Add`가 왜 Race Condition을 만들 수 있는가?
3. `GetOrAdd`는 어떤 문제를 줄여주는가?
4. `GetOrAdd`의 value factory에 결제 API 호출 같은 Side Effect를 넣으면 위험한 이유는?
5. Thread-safe와 Atomic Operation은 어떻게 다른가?
6. `ConcurrentQueue<T>`와 bounded `Channel<T>`의 실무적 차이는 무엇인가?
7. Concurrent Collection을 쓴다고 Backpressure가 자동으로 해결되지 않는 이유는?
8. `ConcurrentDictionary` 기반 Rate Limiter가 여러 App Instance에서 정확하지 않을 수 있는 이유는?
9. Thread-safe와 Distributed-safe의 차이를 설명하라.
10. Concurrent Collection을 쓰더라도 추가 `lock`이 필요할 수 있는 사례를 하나 들어라.
11. 캐시에서 동일 Key에 대한 expensive load가 동시에 여러 번 실행되는 문제를 어떻게 줄일 수 있는가?
12. 60초 안에 Concurrent Collection의 장점과 한계를 설명하라.

## 평가 기준

- Thread-safe와 business-level atomicity를 구분한다.
- `GetOrAdd` factory 중복 실행 가능성을 설명한다.
- Process-local concurrency와 distributed coordination을 구분한다.
- Queue capacity / backpressure와 연결한다.

## 평가 결과

- 점수: Pending
- 보완할 개념: Pending
- 재시험: Pending
