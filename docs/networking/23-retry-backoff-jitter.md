# 23. Retry, Exponential Backoff and Jitter

## Retry는 왜 필요한가

분산 시스템에서는 일시적인 실패가 흔합니다.

예:
- 순간 Packet Loss
- 짧은 연결 끊김
- 일시적인 503
- Leader Election 중 잠깐의 unavailable

이런 transient failure는 Retry 한 번으로 성공할 수 있습니다.

하지만 Retry는 공짜가 아닙니다.

## Retry의 위험

원래 요청이 100개인데 모두 3번씩 재시도하면 최대 300개의 추가 요청이 생길 수 있습니다.

특히 Client → Gateway → Service A → Service B가 각자 Retry하면 Retry Amplification이 발생합니다.

```text
3 retries × 3 retries × 3 retries
= 최대 27배 시도 가능
```

장애 난 시스템에 더 많은 부하를 주는 최악의 상황이 될 수 있습니다.

## 어떤 실패를 Retry할까

Retry에 적합한 예:
- timeout
- 일부 5xx
- connection reset
- 일시적인 unavailable

보통 Retry하면 안 되는 예:
- validation error
- authentication failure
- 대부분의 4xx
- business rule violation

## Idempotency

Retry 전에 반드시 생각해야 하는 질문:

> 같은 요청을 두 번 보내도 안전한가?

GET은 일반적으로 idempotent지만 결제 생성 같은 POST는 그렇지 않을 수 있습니다.

그래서 결제 API는 `Idempotency-Key`를 사용해 중복 요청이 같은 business operation으로 처리되도록 설계할 수 있습니다.

## Immediate Retry의 문제

서버가 과부하 상태인데 즉시 재시도하면 모든 Client가 동시에 다시 몰립니다.

이를 완화하기 위해 Backoff를 사용합니다.

## Exponential Backoff

재시도 간격을 점점 늘립니다.

```text
100ms
200ms
400ms
800ms
```

장애 시스템이 회복할 시간을 줍니다.

## Jitter

모든 Client가 정확히 100, 200, 400ms에 재시도하면 다시 동시에 몰릴 수 있습니다.

이를 Thundering Herd라고 볼 수 있습니다.

그래서 임의의 randomness를 추가합니다.

예:

```text
base delay = 400ms
actual delay = random(0, 400ms)
```

이것이 Jitter입니다.

## Retry Budget

무한 Retry는 안 됩니다.

보통 다음을 제한합니다.
- 최대 Retry 횟수
- 전체 Deadline
- Retryable status
- Retry Traffic 비율

즉 "3번 재시도"만 정하는 것이 아니라 전체 요청 Budget 안에서 관리해야 합니다.

## Retry-After

Server가 `429` 또는 일부 `503` 상황에서 `Retry-After`를 주면 Client가 그 정보를 존중하는 것이 좋습니다.

## Backend 면접 연결

질문: "외부 API가 가끔 503을 반환합니다. 어떻게 하시겠습니까?"

좋은 답변:

```text
Transient failure인지 확인
→ 요청이 idempotent한지 확인
→ 제한된 Retry
→ Exponential Backoff + Jitter
→ 전체 Timeout Budget 준수
→ 지속 실패 시 Circuit Breaker
```

## 흔한 오해

### "Retry는 성공률을 높이므로 많을수록 좋다"

아닙니다. 장애 시 Retry Storm으로 시스템을 더 망가뜨릴 수 있습니다.

### "POST는 절대 Retry하면 안 된다"

Idempotency Key나 business-level deduplication이 있으면 안전하게 Retry하도록 설계할 수 있습니다.

## 60초 면접 답변

Retry는 일시적인 네트워크 오류나 503 같은 transient failure를 복구하는 데 유용하지만 잘못 사용하면 장애 시스템에 추가 부하를 주는 Retry Storm을 만들 수 있습니다. 따라서 retryable error를 제한하고, 요청의 idempotency를 확인하며, 최대 횟수와 전체 deadline을 설정해야 합니다. 재시도 간격은 Exponential Backoff로 늘리고 여러 Client가 동시에 다시 몰리는 것을 막기 위해 Jitter를 추가합니다. 여러 계층이 독립적으로 Retry하면 Retry Amplification이 생길 수 있으므로 Retry 책임을 명확히 두는 것도 중요합니다.