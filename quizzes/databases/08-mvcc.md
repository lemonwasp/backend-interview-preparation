# 08. MVCC — Knowledge Check

## 기본 확인

1. MVCC의 목적은 무엇인가?
2. Row의 여러 Version을 유지하는 이유는?
3. Snapshot은 DB 전체 복사본인가?
4. MVCC에서 Reader와 Writer의 충돌이 줄어드는 이유는?
5. MVCC가 Lock을 완전히 없애는가?

## 구현과 개념

6. Isolation Level과 MVCC의 차이를 설명하라.
7. PostgreSQL과 InnoDB의 Row Version 관리 방식이 왜 완전히 같다고 말하면 안 되는가?
8. 오래 열린 Transaction이 Version Cleanup에 어떤 영향을 줄 수 있는가?
9. PostgreSQL VACUUM이나 InnoDB purge가 MVCC와 어떤 관계가 있는가?
10. 같은 Row를 두 Writer가 수정하면 MVCC에서도 왜 충돌할 수 있는가?

## 실무

11. 외부 HTTP 호출을 DB Transaction 안에서 오래 수행하는 것이 위험한 이유는?
12. MVCC가 Lost Update를 항상 자동으로 막는다고 말할 수 없는 이유는?
13. Optimistic Lock의 version column은 어떤 원리로 충돌을 감지하는가?
14. `UPDATE ... WHERE id=? AND version=?`의 affected rows가 0이면 무엇을 의미할 수 있는가?
15. Batch 작업에서 Long Transaction을 피해야 하는 이유는?

## 면접 답변

16. MVCC를 60초 안에 설명하라.
17. "MVCC면 Lock이 필요 없나요?"에 답하라.
18. "MVCC와 Repeatable Read는 같은 건가요?"에 답하라.
19. "왜 오래 열린 Transaction이 문제인가요?"에 답하라.
20. MVCC, Isolation Level, Optimistic Lock 세 개념의 역할을 각각 구분해 설명하라.

## 평가 기준

- MVCC를 Version + Snapshot + Visibility로 설명할 수 있다.
- MVCC와 Isolation Level을 같은 개념으로 취급하지 않는다.
- Writer-Writer 충돌과 Long Transaction 비용을 설명할 수 있다.
- DBMS별 구현 차이를 과도하게 일반화하지 않는다.

상태: 답변 대기
