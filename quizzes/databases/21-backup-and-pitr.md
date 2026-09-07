# Quiz — Backup / PITR

Status: **답변 대기**

## Questions

1. Replication과 Backup의 목적 차이는 무엇인가요?
2. Replica가 있어도 Backup이 필요한 이유는 무엇인가요?
3. Full / Incremental / Differential Backup의 차이를 설명해보세요.
4. PITR은 어떤 원리로 특정 시점까지 복구하나요?
5. WAL 또는 Transaction Log가 PITR에서 어떤 역할을 하나요?
6. RPO와 RTO의 차이는 무엇인가요?
7. RPO 5분이라는 말은 무슨 뜻인가요?
8. Backup 파일이 존재하는 것만으로 충분하지 않은 이유는 무엇인가요?
9. 잘못된 DELETE가 발생했을 때 Replica Failover만으로 해결되지 않을 수 있는 이유는 무엇인가요?
10. 정기 Restore Test가 중요한 이유는 무엇인가요?
11. Backup 데이터의 암호화와 별도 보관이 필요한 이유는 무엇인가요?
12. Application Rollback과 Database Restore는 왜 다른 문제인가요?

## Interview Drill

> "우리 서비스는 Multi-AZ Replica가 있으니 Backup은 필요 없지 않나요?"

60초 안에 반박해보세요.

## 평가 기준

- Replication = HA/Read Scale, Backup = Historical Recovery를 구분하는가
- PITR = Base Backup + Log Replay를 설명하는가
- RPO/RTO를 정확히 구분하는가
- Restore Test의 필요성을 언급하는가
- 논리적 오류도 Replica로 전파될 수 있음을 설명하는가

## Evaluation

- 정확도: Pending
- 실무 연결: Pending
- 꼬리질문 대응: Pending
- 재시험 필요 여부: Pending
