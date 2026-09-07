# 21. Rate Limiting - Knowledge Check

## Feynman Questions

1. Rate Limiting이 필요한 이유를 서버 보호 관점에서 설명하세요.
2. Fixed Window의 경계 문제는 무엇인가요?
3. Sliding Window는 왜 더 정확하지만 더 비쌀 수 있나요?
4. Token Bucket은 Burst를 어떻게 허용하나요?
5. Token Bucket과 Leaky Bucket의 차이는 무엇인가요?
6. IP만 Rate Limit Key로 사용할 때 어떤 문제가 생길 수 있나요?
7. 분산 서버에서 각 서버가 독립 Counter를 가지면 왜 전체 제한을 어길 수 있나요?
8. 429와 `Retry-After`의 의미를 설명하세요.
9. Rate Limiting과 Throttling의 차이를 설명하세요.
10. Redis 기반 Limiter에도 어떤 병목/장애 가능성이 있나요?

## Interview Drills

11. "초당 1만 요청 API에 사용자당 초당 10회 제한을 구현한다면 어떻게 하시겠습니까?"
12. "Token Bucket을 선택할 상황과 Sliding Window를 선택할 상황을 비교하세요."
13. "Rate Limiting만으로 트래픽 폭주를 해결할 수 없는 이유는 무엇인가요?"
14. 60초 안에 Rate Limiting을 설명하세요.

## Evaluation Criteria

- Fixed/Sliding/Token Bucket 차이를 구분하는가
- Burst와 평균 요청률을 구분하는가
- Distributed 환경의 상태 공유 문제를 이해하는가
- 429/Retry-After를 설명할 수 있는가
- Rate Limiting을 resilience 전체 구조와 연결하는가

## Status

답변 대기