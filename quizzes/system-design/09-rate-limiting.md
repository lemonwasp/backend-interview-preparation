# Quiz — 09. Rate Limiting

## Status

- Explanation: Prepared
- Quiz: Pending
- Re-test: Pending

## Recall Questions

1. Rate Limiting은 어떤 자원을 보호하는가?
2. IP / User / API Key / Tenant 기준의 장단점은 무엇인가?
3. Fixed Window의 boundary burst 문제를 설명하라.
4. Sliding Window의 장점과 비용은 무엇인가?
5. Token Bucket의 capacity와 refill rate는 각각 무엇을 의미하는가?
6. Token Bucket과 Leaky Bucket의 차이를 설명하라.
7. 여러 App Instance에서 local counter만 쓰면 왜 global limit이 깨질 수 있는가?
8. Distributed Rate Limiter가 Redis 같은 shared store를 사용할 때 생기는 비용은 무엇인가?
9. `429 Too Many Requests`와 Retry-After의 역할은 무엇인가?
10. Rate Limit과 Backpressure의 차이는 무엇인가?
11. Rate Limit과 Load Shedding의 차이는 무엇인가?
12. Fail-open / Fail-closed를 어떻게 선택해야 하는가?

## Interview Drills

### Q1
100 req/min 제한 API가 10개 instance로 늘어난 뒤 사용자당 실제 1000 req/min까지 허용되고 있다. 원인은 무엇인가?

### Q2
무료 사용자와 유료 사용자의 quota를 다르게 설계하려면 어떤 key/algorithm을 사용할 수 있는가?

### Q3
Redis 기반 Rate Limiter가 장애 났다. 결제 API와 일반 검색 API의 fail-open/fail-closed 정책을 어떻게 다르게 생각하겠는가?

### Q4
요청 수는 적지만 한 요청이 CPU를 100배 더 쓴다. 단순 request count limit의 한계와 개선 방법을 설명하라.

### Q5
Queue backlog가 커졌을 때 Rate Limiting, Backpressure, Load Shedding을 각각 어떻게 사용할 수 있는가?

## 60-second Test

자료 없이 60초 안에 다음을 포함해 설명한다.

- limiting key
- Token Bucket
- distributed counter
- 429
- Backpressure와 차이
- fail-open / fail-closed
