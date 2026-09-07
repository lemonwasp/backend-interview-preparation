# 06. Transaction Isolation Levels — Knowledge Check

## 기본 확인

1. Isolation Level은 무엇을 조절하는가?
2. Read Uncommitted에서 Dirty Read가 가능한 이유는?
3. Read Committed가 방지하는 대표 이상 현상은?
4. Repeatable Read의 핵심 보장은 무엇인가?
5. Serializable은 실제로 Transaction을 반드시 한 줄씩 실행한다는 뜻인가?

## 이상 현상 연결

6. Dirty Read, Non-repeatable Read, Phantom Read를 각각 한 문장으로 설명하라.
7. 같은 Transaction에서 같은 Row를 두 번 읽었는데 값이 달라졌다. 어떤 현상인가?
8. 같은 조건 조회를 두 번 했더니 조건을 만족하는 Row 수가 늘었다. 어떤 현상인가?
9. SQL 표준 기준으로 Repeatable Read에서 Phantom Read는 어떻게 취급되는가?
10. 같은 Isolation Level 이름이어도 DBMS별 실제 동작이 다를 수 있는 이유는?

## 실무 판단

11. Isolation Level을 무조건 Serializable로 올리는 것이 항상 좋은 설계가 아닌 이유는?
12. 재고 1개를 두 요청이 동시에 구매하려 한다. Isolation Level만 믿지 않고 추가로 고려할 수 있는 방법 세 가지를 말하라.
13. 오래 열린 Transaction이 MVCC 시스템에서 문제가 될 수 있는 이유는?
14. Read-only 통계 화면과 송금 기능에 동일한 동시성 전략이 필요하지 않은 이유는?
15. Lost Update는 Isolation Level만 높이면 항상 해결된다고 말할 수 있는가?

## 면접 답변

16. 60초 안에 네 가지 Isolation Level을 낮은 순서부터 설명하라.
17. "Read Committed와 Repeatable Read의 차이는?"에 답하라.
18. "Repeatable Read면 Phantom Read가 무조건 생기나요?"라는 꼬리질문에 답하라.
19. "Serializable은 왜 비쌀 수 있나요?"에 답하라.
20. 사용하는 DBMS의 기본 Isolation Level과 실제 구현을 왜 확인해야 하는지 설명하라.

## 평가 기준

- 네 Isolation Level을 순서대로 설명할 수 있다.
- Dirty / Non-repeatable / Phantom Read를 혼동하지 않는다.
- SQL 표준과 실제 DBMS 구현을 구분한다.
- Isolation과 Lock/MVCC/Optimistic Lock의 관계를 과도하게 단순화하지 않는다.

상태: 답변 대기
