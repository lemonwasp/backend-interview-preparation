# 10. Database Deadlocks — Knowledge Check

## 기본 확인

1. Database Deadlock이란 무엇인가?
2. Circular Wait를 간단한 두 Transaction 예로 설명하라.
3. Deadlock과 Lock Wait Timeout의 차이는?
4. DBMS는 Deadlock을 감지하면 일반적으로 어떻게 처리하는가?
5. Deadlock Victim이란 무엇인가?

## 예방

6. 일관된 Lock Ordering이 Deadlock 가능성을 줄이는 이유는?
7. 계좌 이체에서 항상 작은 Account ID부터 Lock하는 전략이 왜 유용한가?
8. Long Transaction이 Deadlock 확률을 높이는 이유는?
9. 적절한 Index가 Deadlock 가능성을 낮추는 데 도움을 줄 수 있는 이유는?
10. `SELECT FOR UPDATE` 뒤 외부 API를 호출하는 패턴이 위험한 이유는?

## Retry와 복구

11. Deadlock 오류가 발생했을 때 무조건 무한 Retry하면 안 되는 이유는?
12. Retry 전에 Idempotency를 확인해야 하는 이유는?
13. Deadlock 전에 외부 결제가 성공했다면 Transaction Retry가 왜 위험한가?
14. Outbox / Idempotency Key 같은 패턴이 어떤 문제를 줄이는 데 도움을 주는가?
15. Deadlock 발생률이 갑자기 증가했다면 어떤 항목들을 조사할 것인가?

## OS와 연결

16. OS Deadlock과 DB Deadlock의 공통 원리는 무엇인가?
17. Coffman Conditions 네 가지를 말하라.
18. Database에서는 어떤 자원이 Deadlock에 참여할 수 있는가?

## 면접 답변

19. "Deadlock은 DB 버그인가요?"에 답하라.
20. Database Deadlock을 60초 안에 설명하라.

## 평가 기준

- Deadlock과 단순 Lock Wait를 구분한다.
- Victim Abort + Retry 복구 흐름을 설명할 수 있다.
- Lock Ordering, Short Transaction, Index를 예방책으로 연결한다.
- 외부 Side Effect와 Retry의 위험을 설명할 수 있다.

상태: 답변 대기
