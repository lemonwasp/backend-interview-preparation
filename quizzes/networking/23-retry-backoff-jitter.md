# 23. Retry / Backoff / Jitter - Knowledge Check

## Feynman Questions

1. Retry가 필요한 transient failure의 예를 세 가지 드세요.
2. Retry Amplification은 어떻게 발생하나요?
3. 어떤 4xx/5xx를 Retry해야 하는지 판단할 때 무엇을 봐야 하나요?
4. Idempotency가 Retry에서 왜 중요한가요?
5. `Idempotency-Key`는 어떤 문제를 해결하나요?
6. Immediate Retry가 장애를 악화시킬 수 있는 이유는 무엇인가요?
7. Exponential Backoff는 무엇인가요?
8. Jitter가 없으면 어떤 문제가 생길 수 있나요?
9. Retry Budget에 포함할 요소를 설명하세요.
10. `Retry-After`를 언제 사용할 수 있나요?

## Interview Drills

11. "외부 결제 API가 가끔 503을 반환한다면 어떻게 재시도하겠습니까?"
12. "Client, Gateway, Service가 모두 3회 Retry하면 어떤 문제가 생길 수 있나요?"
13. "POST 요청도 안전하게 Retry할 수 있는 설계를 설명하세요."
14. 60초 안에 Retry / Backoff / Jitter를 설명하세요.

## Evaluation Criteria

- transient/permanent failure를 구분하는가
- retry amplification을 이해하는가
- idempotency와 retry를 연결하는가
- exponential backoff와 jitter의 목적을 구분하는가
- timeout/retry budget을 함께 고려하는가

## Status

답변 대기