# Quiz — 10. Distributed Lock

## Status

- Explanation: Prepared
- Quiz: Pending
- Re-test: Pending

## Recall Questions

1. Process-local `lock`이 여러 App Instance 간 race를 막지 못하는 이유는 무엇인가?
2. Distributed Lock을 도입하기 전에 어떤 더 단순한 대안을 먼저 검토해야 하는가?
3. Lease / TTL이 필요한 이유는 무엇인가?
4. TTL이 있어도 stale owner 문제가 생길 수 있는 이유는 무엇인가?
5. Fencing Token은 어떤 문제를 해결하는가?
6. Fencing Token은 왜 downstream resource의 지원이 필요한가?
7. Lock 획득의 `존재 확인 → 생성` 패턴이 race-safe하지 않은 이유는 무엇인가?
8. Distributed Lock이 Transaction이나 Exactly-once를 자동 보장하지 않는 이유는 무엇인가?
9. Distributed Lock과 Idempotency의 차이는 무엇인가?
10. Lock Granularity가 너무 크거나 너무 작으면 각각 어떤 문제가 생기는가?
11. 여러 Distributed Lock을 동시에 잡을 때 Deadlock 가능성을 어떻게 줄일 수 있는가?
12. Coordination Store 장애 시 fail-safe 정책을 어떻게 결정해야 하는가?

## Interview Drills

### Q1
매일 한 번 정산 Job이 여러 Worker에서 동시에 시작될 수 있다. 어떤 방식으로 중복 실행을 막겠는가?

### Q2
Redis TTL lock을 사용 중인데 드물게 동일 작업이 두 번 실행된다. 어떤 stale-owner 시나리오를 의심할 수 있는가?

### Q3
마지막 재고 1개를 보호하기 위해 Distributed Lock을 도입하려 한다. 더 단순한 DB 방식이 있는가?

### Q4
Lock을 획득한 Worker가 DB update 후 외부 API 호출에서 timeout됐다. Lock이 이 문제를 모두 해결해주는가?

### Q5
Distributed Lock Service가 장애 났다. 중복 실행이 치명적인 정산과, 중복 실행돼도 비용만 드는 cache rebuild의 정책을 어떻게 다르게 잡겠는가?

## 60-second Test

자료 없이 60초 안에 다음을 포함해 설명한다.

- process-local vs distributed
- atomic acquisition
- lease / TTL
- stale owner
- fencing token
- idempotency와 차이
- DB primitive 우선 검토
