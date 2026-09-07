# 25. Bulkhead / Resilience Patterns - Knowledge Check

## Feynman Questions

1. Bulkhead라는 이름이 어디서 왔고 시스템에서는 어떤 의미인가요?
2. 느린 dependency가 전체 Thread Pool을 고갈시키는 과정을 설명하세요.
3. Bulkhead를 구현하는 방법을 네 가지 이상 말하세요.
4. Semaphore Bulkhead는 어떻게 동작하나요?
5. Concurrency Limit만 있고 Queue가 무한하면 어떤 문제가 생기나요?
6. Backpressure란 무엇인가요?
7. Load Shedding과 Graceful Degradation의 차이는 무엇인가요?
8. Rate Limit / Timeout / Retry / Circuit Breaker / Bulkhead의 역할을 각각 한 문장으로 설명하세요.
9. Connection Pool Exhaustion을 Bulkhead 관점에서 어떻게 완화할 수 있나요?
10. Resilience가 '절대 실패하지 않는 것'이 아닌 이유는 무엇인가요?

## Interview Drills

11. "추천 서비스 장애 때문에 쇼핑몰 전체가 느려집니다. 어떻게 격리하시겠습니까?"
12. "모든 dependency가 같은 Thread Pool과 Connection Pool을 공유하면 어떤 위험이 있나요?"
13. "Queue를 추가하면 과부하가 해결된다는 주장에 반박해 보세요."
14. 60초 안에 Bulkhead와 대표 Resilience Pattern을 설명하세요.

## Evaluation Criteria

- 자원 격리의 목적을 이해하는가
- bounded concurrency와 bounded queue를 함께 고려하는가
- backpressure/load shedding을 설명하는가
- resilience pattern들의 역할을 구분하는가
- 장애 시 핵심 기능 우선순위를 생각하는가

## Status

답변 대기