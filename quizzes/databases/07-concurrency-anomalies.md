# 07. Concurrency Anomalies — Knowledge Check

## 기본 확인

1. Dirty Read란 무엇인가?
2. Non-repeatable Read란 무엇인가?
3. Phantom Read란 무엇인가?
4. Lost Update란 무엇인가?
5. Write Skew는 어떤 유형의 불변식 문제인가?

## 구분 문제

6. 같은 Row를 두 번 읽었는데 값이 달라졌다. 어떤 현상인가?
7. 같은 WHERE 조건으로 조회했는데 Row 수가 늘었다. 어떤 현상인가?
8. 두 요청이 stock=10을 읽고 둘 다 9를 저장해 최종값이 9가 되었다. 어떤 현상인가?
9. 아직 COMMIT하지 않은 balance=0을 다른 Transaction이 읽었다. 어떤 현상인가?
10. 서로 다른 Row를 수정했지만 전체 업무 규칙이 깨졌다. 어떤 현상을 의심할 수 있는가?

## SQL 설계

11. `SELECT stock` 후 Application에서 -1해서 `UPDATE stock = 9` 하는 방식이 위험한 이유는?
12. `UPDATE products SET stock = stock - 1 WHERE stock > 0`이 더 안전할 수 있는 이유는?
13. 중복 예약을 막을 때 `SELECT` 존재 확인만으로 부족한 이유는?
14. UNIQUE Constraint가 동시성 상황에서 어떤 역할을 하는가?
15. 좋아요 수 증가에서 Application read-modify-write보다 Atomic Increment가 유리한 이유는?

## 면접 꼬리질문

16. Non-repeatable Read와 Phantom Read의 차이를 30초 안에 설명하라.
17. "Transaction으로 감싸면 Lost Update가 사라지나요?"에 답하라.
18. "Application에서 중복 체크하면 UNIQUE Constraint가 왜 필요하죠?"에 답하라.
19. Lost Update를 막는 방법 세 가지를 말하라.
20. Write Skew가 단순 Row Lock만으로 해결되지 않을 수 있는 이유를 설명하라.

## 평가 기준

- Dirty / Non-repeatable / Phantom을 정확히 구분한다.
- Lost Update를 실제 Backend 패턴과 연결한다.
- Check-then-act 경쟁 조건을 설명할 수 있다.
- DB Constraint를 동시성 방어선으로 이해한다.

상태: 답변 대기
