# Quiz — Database Failure Scenarios

Status: **답변 대기**

## Questions

1. API latency가 급증했는데 DB CPU는 낮습니다. 어떤 원인을 먼저 의심할 수 있나요?
2. Connection Pool Exhaustion의 대표 원인과 관찰 지표를 말해보세요.
3. Pool Size를 무작정 키우면 왜 문제가 악화될 수 있나요?
4. Lock Contention과 Slow Query를 어떻게 구분해볼 수 있나요?
5. Deadlock과 일반 Lock Wait의 차이는 무엇인가요?
6. Execution Plan에서 어떤 항목을 확인해야 하나요?
7. DB CPU가 낮아도 Storage Saturation일 수 있는 이유는 무엇인가요?
8. Replica Lag의 대표 증상과 대응 방법은 무엇인가요?
9. Failover 후 Client timeout이 발생했는데 실제 Transaction은 Commit됐을 수도 있다는 말은 무슨 뜻인가요?
10. 이 상황에서 Idempotency가 왜 중요한가요?
11. Split-brain이란 무엇이며 왜 위험한가요?
12. Replica가 Logical Corruption을 막아주지 못하는 이유는 무엇인가요?
13. Cache Stampede가 DB 장애를 어떻게 증폭시킬 수 있나요?
14. 장애 대응에서 Scope → Metrics → Mitigation → Root Cause 순서가 중요한 이유는 무엇인가요?
15. 최근 Schema Migration 직후 전체 Write latency가 증가했다면 무엇을 확인하겠습니까?

## Scenario Drill

### Scenario A

- API p99 latency 급증
- DB CPU 35%
- Connection acquire latency 급증
- Lock wait 증가

원인 가설과 대응을 말해보세요.

### Scenario B

- 사용자 주문 생성 성공
- 바로 조회하면 주문이 안 보임
- 3초 후 보임

원인과 해결 방향을 말해보세요.

### Scenario C

- DB Failover 직후 Client가 timeout
- 같은 결제를 Retry하려고 함

어떤 위험이 있고 어떻게 설계해야 하나요?

## 평가 기준

- 증상과 원인을 구분하는가
- Application/DB/Storage 지표를 연결하는가
- Pool/Lock/Replica/Failover를 구분하는가
- Retry와 Idempotency를 함께 고려하는가
- 즉시 Mitigation과 근본 원인을 구분하는가

## Evaluation

- 정확도: Pending
- 장애 대응 구조화: Pending
- 꼬리질문 대응: Pending
- 재시험 필요 여부: Pending
