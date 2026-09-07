# 12. Observability / SLO — Knowledge Check

상태: 답변 대기

## 기본 질문

1. Monitoring과 Observability의 차이를 설명해보세요.
2. Metrics, Logs, Traces는 각각 어떤 질문에 가장 잘 답하나요?
3. RED Method의 세 항목은 무엇인가요?
4. USE Method는 어떤 종류의 병목을 찾는 데 유용한가요?
5. SLI, SLO, SLA의 차이를 설명해보세요.
6. 99.9% availability가 의미하는 월간 downtime을 대략 계산해보세요.
7. Error Budget은 무엇이고 왜 필요한가요?
8. 평균 latency보다 p95/p99가 중요한 이유는 무엇인가요?

## 꼬리 질문

9. CPU 90%인데 사용자 latency는 정상입니다. 즉시 장애라고 판단해야 하나요?
10. Metric label에 user_id를 넣으면 어떤 문제가 생길 수 있나요?
11. Queue 기반 시스템에서는 어떤 metric을 보면 좋을까요?
12. DB 문제를 진단할 때 query latency 외에 어떤 지표를 같이 보겠습니까?
13. Trace ID가 분산 시스템에서 중요한 이유는 무엇인가요?
14. 모든 요청을 100% trace하면 왜 문제가 될 수 있나요?
15. SLO를 100%로 잡는 것이 왜 항상 좋은 목표가 아닌가요?

## 시나리오

16. API 평균 latency는 100ms인데 고객이 느리다고 합니다. 어떤 지표를 먼저 보겠습니까?
17. 결제 성공률 SLO가 악화됐습니다. Metric → Trace → Log 순으로 어떻게 조사하겠습니까?
18. Error Budget을 이번 달에 거의 소진했습니다. 배포 정책을 어떻게 바꾸겠습니까?

## 60초 면접 답변

19. "좋은 Observability 시스템은 어떻게 설계합니까?"에 60초 안에 답해보세요.

## 평가 기준

- Metric/Log/Trace의 역할을 구분하는가
- RED/USE를 실제 병목 진단과 연결하는가
- SLI/SLO/SLA/Error Budget을 혼동하지 않는가
- 평균이 아니라 tail latency와 saturation을 보는가
- Observability 자체의 비용/cardinality/sampling trade-off를 설명하는가
