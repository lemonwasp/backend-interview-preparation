# System Design Final Mock Interview

> 상태: 답변 대기  
> 목표: 50문항 중 최소 40문항에서 핵심 개념 혼동 없이 답변  
> 기준: 요구사항 → 규모 → 설계 → trade-off → failure mode 순서

## A. Requirements / Estimation

1. System Design 면접에서 기술 선택보다 먼저 확인해야 할 것은 무엇인가요?
2. 평균 RPS만 보고 설계하면 왜 위험한가요?
3. Read/Write ratio가 설계에 어떤 영향을 줍니까?
4. Storage growth와 bandwidth를 왜 대략이라도 계산해야 하나요?
5. Availability, latency, consistency 요구를 모두 최대로 잡으면 왜 문제가 되나요?

## B. Stateless / Scaling

6. Stateless Service가 horizontal scaling에 유리한 이유는 무엇인가요?
7. Sticky Session의 장점과 단점은 무엇인가요?
8. App instance를 10배로 늘렸는데 성능이 오르지 않는 이유를 어떻게 찾겠습니까?
9. `instance count × DB pool size`가 중요한 이유는 무엇인가요?
10. Readiness와 connection draining은 배포 중 어떤 문제를 줄이나요?

## C. Cache

11. Cache-Aside를 설명해보세요.
12. Cache invalidation이 실패하면 어떤 문제가 생기나요?
13. Cache Stampede와 Hot Key의 차이는 무엇인가요?
14. Local Cache와 Distributed Cache를 언제 각각 선택하나요?
15. Cache 장애가 DB 장애로 전파되는 시나리오를 설명해보세요.

## D. Queue / Async Processing

16. Message Queue를 사용하는 가장 큰 이유는 무엇인가요?
17. At-least-once delivery에서 왜 consumer idempotency가 필요하나요?
18. Queue depth가 증가하면 consumer를 무조건 늘리면 안 되는 이유는 무엇인가요?
19. DLQ의 역할과 한계는 무엇인가요?
20. Sync API와 Async Queue를 선택하는 기준을 설명해보세요.

## E. Database Scaling / Consistency

21. Read-heavy DB를 확장하는 방법을 우선순위대로 설명해보세요.
22. Read Replica가 만드는 대표적인 consistency 문제는 무엇인가요?
23. Write-heavy 시스템에서 고려할 수 있는 확장 전략은 무엇인가요?
24. CAP를 'C/A/P 중 두 개를 고르는 것'이라고만 설명하면 왜 부정확한가요?
25. Strong Consistency와 Eventual Consistency를 각각 어떤 데이터에 적용하겠습니까?

## F. Idempotency / Rate Limit / Lock

26. 결제 API가 timeout되어 client가 retry했습니다. 중복 결제를 어떻게 막겠습니까?
27. Idempotency Key에 같은 key지만 다른 payload가 오면 어떻게 해야 하나요?
28. Rate Limiting, Backpressure, Load Shedding의 차이를 설명해보세요.
29. Token Bucket이 Fixed Window보다 유리한 점은 무엇인가요?
30. Distributed Lock보다 DB UNIQUE constraint나 conditional update를 먼저 검토해야 하는 이유는 무엇인가요?
31. Lease 기반 Distributed Lock의 stale owner 문제는 무엇인가요?
32. Fencing Token은 어떤 문제를 해결하나요?

## G. Service Boundary / Ownership

33. Service Boundary를 나누는 기준은 무엇인가요?
34. Shared DB가 주는 장점과 위험은 무엇인가요?
35. Database per Service가 왜 distributed consistency 비용을 만드나요?
36. 어떤 상황에서 Microservice보다 Modular Monolith가 더 적절합니까?
37. 다른 서비스가 Owner DB를 직접 수정하면 왜 문제가 되나요?

## H. Observability / SLO

38. Metrics, Logs, Traces의 역할 차이를 설명해보세요.
39. RED와 USE는 각각 무엇을 보기 위한 관점인가요?
40. SLI, SLO, SLA의 차이를 설명해보세요.
41. Error Budget은 왜 필요한가요?
42. 평균 latency보다 p95/p99가 중요한 이유는 무엇인가요?

## I. Reliability / Failure Mode

43. Graceful Degradation의 구체적인 예를 하나 들어보세요.
44. Dependency가 느려졌을 때 retry가 장애를 악화시킬 수 있는 이유는 무엇인가요?
45. Failure Mode를 분석할 때 down보다 slow/partial failure가 중요한 이유는 무엇인가요?
46. Write timeout이 발생했을 때 commit 여부가 불명확하면 어떻게 처리하겠습니까?
47. Cache outage → DB overload → 전체 장애로 이어지는 경로를 끊는 방법을 설명해보세요.

## J. Multi-region / DR

48. Active-Passive와 Active-Active를 비교해보세요.
49. RPO와 RTO는 각각 무엇이며 설계에 어떻게 반영합니까?
50. Region 전체가 장애났을 때 traffic switch 외에 반드시 확인해야 할 항목을 설명해보세요.

---

# Evaluation Rubric

각 답변을 0~2점으로 평가한다.

- 0점: 핵심 개념 오류 또는 답변 불가
- 1점: 정의는 맞지만 trade-off / failure mode 연결 부족
- 2점: 정의 + 이유 + trade-off + backend 사례까지 설명

총점 100점.

## 최소 통과 기준

- 80점 이상
- 50문항 중 최소 40문항 핵심 개념 혼동 없음
- 다음 비교 질문 필수 통과
  - Strong vs Eventual Consistency
  - Idempotency vs Distributed Lock
  - Rate Limit vs Backpressure vs Load Shedding
  - Replication vs Sharding
  - Active-Passive vs Active-Active
  - Microservice vs Modular Monolith
- 다음 장애 시나리오 필수 통과
  - Cache outage
  - Queue backlog
  - DB latency spike
  - Retry storm
  - Ambiguous write timeout
  - Region failure

## Answer Structure

설계형 질문은 가능하면 다음 순서로 답한다.

1. Requirement
2. Scale
3. Main bottleneck
4. Proposed design
5. Consistency / duplicate handling
6. Failure mode
7. Observability
8. Trade-off

## Status

- Mock interview: Pending
- Re-test: Pending
- Completed: No
