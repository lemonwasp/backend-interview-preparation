# 22. Timeout Budget - Knowledge Check

## Feynman Questions

1. Timeout이 없으면 느린 dependency가 서버 자원을 어떻게 고갈시킬 수 있나요?
2. Connection Timeout과 Read Timeout은 어떻게 다른가요?
3. Timeout Budget이란 무엇인가요?
4. downstream timeout이 upstream 전체 timeout보다 작아야 하는 이유는 무엇인가요?
5. Deadline Propagation이 필요한 이유를 설명하세요.
6. 너무 긴 Timeout과 너무 짧은 Timeout의 문제를 각각 말하세요.
7. Timeout과 Retry를 함께 사용할 때 전체 latency budget을 왜 계산해야 하나요?
8. 상위 요청이 취소됐는데 downstream 작업이 계속되면 어떤 문제가 생기나요?
9. p95/p99 latency가 Timeout 설정에 왜 중요한가요?
10. 모든 dependency에 같은 Timeout을 쓰면 안 되는 이유는 무엇인가요?

## Interview Drills

11. "사용자 SLA가 2초이고 서비스가 DB와 외부 API를 호출한다면 timeout budget을 어떻게 나누겠습니까?"
12. "결제 API timeout을 정할 때 어떤 정보를 확인하시겠습니까?"
13. "Timeout이 cascading failure를 줄이는 이유를 설명하세요."
14. 60초 안에 Timeout Budget을 설명하세요.

## Evaluation Criteria

- timeout 종류를 구분하는가
- 전체 deadline과 하위 timeout의 관계를 이해하는가
- cancellation과 wasted work를 연결하는가
- retry와 latency budget을 함께 고려하는가
- 임의의 숫자보다 측정과 SLO를 기준으로 답하는가

## Status

답변 대기