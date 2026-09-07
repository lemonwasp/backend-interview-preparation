# 24. Circuit Breaker

## 왜 필요한가

Downstream Service가 계속 실패하는데 매 요청마다 끝까지 호출하면 다음 문제가 생깁니다.

- Timeout 대기 증가
- Thread / Connection 점유
- Retry Storm
- Tail latency 증가
- Cascading Failure

Circuit Breaker는 일정 수준 이상 실패가 계속되면 잠시 호출 자체를 차단합니다.

전기 회로의 차단기처럼 장애 구간을 빠르게 격리하는 것입니다.

## 세 가지 상태

### Closed

정상 상태입니다.

요청을 통과시키고 성공/실패를 관찰합니다.

### Open

실패율이 임계치를 넘으면 회로를 엽니다.

이 상태에서는 downstream을 실제로 호출하지 않고 빠르게 실패시킵니다.

이를 Fail Fast라고 합니다.

### Half-Open

일정 시간이 지난 뒤 소수의 Probe Request만 허용합니다.

- 성공하면 Closed로 복귀
- 다시 실패하면 Open 유지

## 왜 Fail Fast가 중요한가

이미 장애가 난 dependency를 5초씩 기다리는 대신 수 ms 안에 실패하면 서버 자원을 보존할 수 있습니다.

그리고 fallback이나 degraded mode를 더 빨리 선택할 수 있습니다.

## Failure Threshold

Circuit을 언제 열지는 정책으로 정합니다.

예:

```text
최근 100개 중 실패율 50% 이상
minimum requests = 20
open duration = 30s
```

단순 연속 실패 횟수보다 sliding window와 minimum sample을 함께 사용하는 경우가 많습니다.

## 어떤 실패를 집계할까

모든 실패를 Circuit Breaker 실패로 세면 안 됩니다.

예:
- 400 validation error -> downstream 장애가 아닐 수 있음
- 401 -> 인증 문제
- 500 / timeout / connection failure -> dependency 장애 신호일 가능성 큼

## Circuit Breaker와 Retry

둘은 반대가 아니라 함께 사용될 수 있습니다.

```text
짧은 Retry
→ 계속 실패
→ Circuit Open
→ Fail Fast
```

하지만 Retry를 과도하게 넣으면 Circuit이 열리기 전에 부하를 더 키울 수 있습니다.

## Circuit Breaker와 Timeout

Circuit Breaker가 Timeout을 대체하지는 않습니다.

- Timeout: 개별 호출을 얼마나 기다릴지
- Circuit Breaker: 반복되는 실패 상황에서 호출 자체를 차단할지

둘 다 필요할 수 있습니다.

## Fallback

Open 상태에서 가능한 대응:

- Cache 반환
- 기본값 반환
- 일부 기능 비활성화
- Queue에 적재 후 나중 처리
- 명확한 오류 반환

모든 서비스에 fallback이 가능한 것은 아닙니다.

결제 승인처럼 임의의 기본값을 반환하면 안 되는 경우도 있습니다.

## 위험: Breaker 자체의 잘못된 설정

너무 민감하면 정상적인 짧은 장애에도 Circuit이 자주 열립니다.

너무 둔하면 장애 격리가 늦습니다.

따라서 다음을 관찰해야 합니다.

- failure rate
- timeout rate
- latency
- open count
- half-open success rate

## Instance-local vs Shared

Circuit Breaker 상태는 보통 각 Application Instance 내부에 둘 수 있습니다.

모든 Instance가 정확히 같은 상태를 공유해야 하는 것은 아닙니다.

오히려 중앙 상태 저장소에 강하게 의존하면 Breaker 자체가 추가 dependency가 될 수 있습니다.

## Backend 면접 연결

질문: "외부 추천 API가 80% timeout인데 서비스 전체가 느려졌습니다. 어떻게 대응하시겠습니까?"

```text
짧은 timeout
→ 제한된 retry
→ circuit breaker
→ fallback/cache
→ metrics/alert
```

처럼 답할 수 있습니다.

## 흔한 오해

### "Circuit Breaker는 장애를 고친다"

아닙니다. 장애를 격리하고 실패를 빠르게 만들어 시스템 전체 피해를 줄이는 패턴입니다.

### "Circuit이 Open이면 모든 기능이 정상이다"

아닙니다. 서비스는 degraded mode일 수 있습니다.

## 60초 면접 답변

Circuit Breaker는 downstream dependency가 반복적으로 실패할 때 계속 호출하지 않고 일정 시간 호출을 차단해 cascading failure를 줄이는 resilience pattern입니다. 정상일 때는 Closed 상태이고 실패율이 임계치를 넘으면 Open되어 fail fast합니다. 이후 일정 시간이 지나면 Half-Open에서 일부 probe 요청을 보내 회복 여부를 확인합니다. Timeout은 개별 호출의 최대 대기 시간을 제한하고, Circuit Breaker는 반복되는 실패에서 호출 자체를 막는다는 차이가 있습니다. 필요하면 cache나 degraded response 같은 fallback과 함께 사용합니다.