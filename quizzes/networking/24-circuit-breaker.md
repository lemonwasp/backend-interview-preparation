# 24. Circuit Breaker - Knowledge Check

## Feynman Questions

1. Circuit Breaker가 필요한 이유를 cascading failure 관점에서 설명하세요.
2. Closed / Open / Half-Open 상태를 설명하세요.
3. Open 상태에서 fail fast가 자원 보호에 왜 도움이 되나요?
4. Failure Threshold를 정할 때 sample size가 왜 중요한가요?
5. 모든 4xx/5xx를 같은 실패로 집계하면 안 되는 이유는 무엇인가요?
6. Circuit Breaker와 Retry를 어떻게 함께 사용할 수 있나요?
7. Circuit Breaker와 Timeout의 차이는 무엇인가요?
8. Fallback의 예를 세 가지 드세요.
9. Fallback이 위험할 수 있는 business operation의 예를 드세요.
10. Circuit Breaker가 장애 자체를 해결하는 기술이 아닌 이유는 무엇인가요?

## Interview Drills

11. "추천 API가 80% timeout이면 서비스 전체가 느려집니다. 어떻게 대응하시겠습니까?"
12. "Circuit Breaker가 너무 민감하거나 너무 둔하면 각각 어떤 문제가 생기나요?"
13. "Breaker 상태를 중앙 Redis에 반드시 공유해야 합니까?"
14. 60초 안에 Circuit Breaker를 설명하세요.

## Evaluation Criteria

- 3-state model을 정확히 설명하는가
- fail fast와 resource protection을 연결하는가
- timeout/retry와 역할을 구분하는가
- fallback의 장단점을 이해하는가
- 설정 임계치와 observability를 고려하는가

## Status

답변 대기