# 15. Multi-region / Disaster Recovery — Knowledge Check

상태: 답변 대기

## 기본 질문

1. Multi-region을 고려하는 대표적인 이유는 무엇인가요?
2. RPO와 RTO의 차이를 설명해보세요.
3. Backup만 있다고 DR이 완성되지 않는 이유는 무엇인가요?
4. Active-Passive와 Active-Active의 차이와 trade-off를 설명해보세요.
5. Single Writer + Multi-region Read 구조의 장점과 한계는 무엇인가요?
6. Region Failover 과정에서 단순 DNS 변경 외에 무엇을 확인해야 하나요?
7. Split Brain이란 무엇이고 왜 위험한가요?
8. Active-Active에서 Data Conflict를 어떻게 처리할 수 있나요?
9. Global Traffic Routing과 Data Locality를 함께 봐야 하는 이유는 무엇인가요?
10. Failback이 왜 별도의 설계를 필요로 하나요?

## 꼬리 질문

11. Tokyo와 Europe 두 Region을 Active-Active로 만들었는데 계좌 잔액을 Last-Write-Wins로 합치자는 제안이 있습니다. 어떻게 평가하겠습니까?
12. App은 두 Region에 있지만 DB는 한 Region에만 있습니다. 어떤 장애에 취약합니까?
13. Cross-region async replication을 사용하는 경우 RPO는 무엇의 영향을 받나요?
14. 자동 failover가 너무 민감하면 어떤 문제가 생길 수 있나요?
15. Region 장애 중 standby가 primary로 승격된 뒤 원래 primary가 다시 살아났습니다. 무엇을 조심해야 하나요?
16. 데이터 주권 요구사항이 Region routing에 어떤 영향을 줄 수 있나요?

## 시나리오

17. 서비스 요구사항이 RPO 5분, RTO 1시간입니다. 처음부터 Active-Active가 필요한지 판단해보세요.
18. Primary Region이 완전히 장애났지만 standby replica가 3분 뒤처져 있습니다. 어떤 판단을 해야 하나요?
19. DR 문서상 RTO는 30분인데 최근 restore test에서 4시간이 걸렸습니다. 무엇이 문제인가요?
20. Region 복구 후 원래 Region으로 traffic을 되돌리려 합니다. Failback 절차를 설명하세요.

## 60초 면접 답변

21. "글로벌 서비스를 Multi-region으로 설계한다면 무엇부터 결정합니까?"에 60초 안에 답해보세요.

## 평가 기준

- RPO/RTO를 architecture decision의 출발점으로 사용하는가
- Active-Passive와 Active-Active를 단순 우열이 아니라 trade-off로 설명하는가
- replication lag, conflict, split brain, fencing을 이해하는가
- failover뿐 아니라 failback/reconciliation을 고려하는가
- DR은 실제 restore/failover drill로 검증해야 한다고 설명하는가
