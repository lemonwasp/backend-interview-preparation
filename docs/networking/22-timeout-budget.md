# 22. Timeout Budget

## 왜 Timeout이 필요한가

외부 서비스나 Database가 느려졌을 때 무한정 기다리면 요청 하나가 Thread, Connection, Memory 같은 자원을 계속 점유합니다.

Timeout은 "이 작업을 언제 포기할지" 정하는 보호 장치입니다.

## Timeout은 하나가 아니다

일반적으로 다음처럼 여러 단계가 있습니다.

- Connection Timeout
- Read Timeout
- Write Timeout
- Request Timeout
- Database Command Timeout
- Upstream Timeout

각 단계의 의미가 다르므로 하나의 큰 Timeout 값만 두는 것은 좋지 않습니다.

## Timeout Budget

사용자 요청 전체가 2초 안에 끝나야 한다고 가정합니다.

```text
Client budget: 2000ms
Gateway: 1800ms
Service A: 1500ms
Service B call: 800ms
DB: 300ms
```

핵심은 downstream Timeout이 upstream 전체 Timeout보다 작아야 한다는 것입니다.

그렇지 않으면 상위 요청은 이미 실패했는데 하위 작업이 계속 실행되는 waste가 발생합니다.

## Deadline Propagation

분산 시스템에서는 단순한 "각 서비스마다 1초"보다 요청 전체의 Deadline을 전달하는 것이 더 좋을 수 있습니다.

예:

```text
remaining budget = 420ms
```

이 상태에서 downstream에게 2초 Timeout을 주는 것은 의미가 없습니다.

gRPC의 Deadline이 대표적인 예입니다.

## 너무 긴 Timeout

- 자원 점유 증가
- Queue 증가
- Tail latency 악화
- 장애 감지 지연
- Cascading Failure 가능성 증가

## 너무 짧은 Timeout

- 정상적으로 처리될 요청까지 실패
- Retry 증가
- 불필요한 Load 증가
- 일시적인 network jitter에도 실패

따라서 p50만 보고 정하는 것이 아니라 p95/p99 latency와 business requirement를 같이 봐야 합니다.

## Timeout과 Retry

Timeout 뒤 Retry를 한다면 총 사용자 Latency Budget 안에 들어와야 합니다.

```text
전체 Budget = 1000ms
1차 시도 = 400ms
Backoff = 100ms
2차 시도 = 400ms
```

이미 900ms입니다.

Retry 횟수만 늘리면 성공률은 올라갈 수 있지만 tail latency와 load도 같이 증가합니다.

## Cancellation

상위 요청이 취소되거나 Timeout되면 가능한 경우 downstream 작업도 취소해야 합니다.

그렇지 않으면 사용자는 이미 떠났는데 서버에서는 계속 DB Query나 외부 API 호출을 수행할 수 있습니다.

## Backend 면접 연결

질문: "외부 결제 API timeout을 몇 초로 잡겠습니까?"

좋은 답변은 임의의 숫자를 말하기보다 다음을 확인합니다.

- 사용자 전체 SLA/SLO
- 해당 API의 정상 latency distribution
- Retry 여부
- Idempotency
- 실패했을 때 business cost
- downstream cancellation 지원 여부

## 흔한 오해

### "Timeout은 길게 잡을수록 안전하다"

오히려 느린 장애에서 자원 고갈과 cascading failure를 키울 수 있습니다.

### "모든 dependency에 같은 Timeout을 쓰면 된다"

Dependency마다 latency 특성과 중요도가 다릅니다.

## 60초 면접 답변

Timeout은 느리거나 응답하지 않는 dependency를 무한정 기다리지 않도록 요청의 최대 대기 시간을 제한하는 resilience 장치입니다. 분산 시스템에서는 각 서비스가 독립적으로 큰 Timeout을 두기보다 전체 요청의 latency budget 안에서 downstream timeout을 더 작게 배분하는 것이 중요합니다. 상위 요청이 이미 timeout된 뒤 하위 작업이 계속되는 낭비를 막기 위해 deadline propagation과 cancellation도 고려해야 합니다. 너무 긴 timeout은 자원 점유와 cascading failure를 키우고, 너무 짧으면 정상 요청까지 실패시킬 수 있으므로 p95/p99 latency와 business SLA를 기준으로 조정해야 합니다.