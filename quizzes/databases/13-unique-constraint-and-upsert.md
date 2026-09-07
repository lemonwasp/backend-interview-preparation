# 13. Unique Constraint and Upsert — Knowledge Check

## 상태

- 답변: Pending
- Re-test: Pending

## 기본 질문

1. Application에서 `SELECT 후 INSERT`만으로 중복을 막을 수 없는 이유는 무엇인가?
2. Unique Constraint가 동시성 환경에서 강력한 이유는 무엇인가?
3. Composite Unique Constraint가 표현할 수 있는 비즈니스 규칙의 예를 들어라.
4. Upsert란 무엇인가?
5. Constraint와 Index를 개념적으로 구분하라.

## 설계 질문

6. 사용자 이메일 중복 방지를 어떻게 설계하겠는가?
7. `(user_id, coupon_id)`를 Unique로 두면 어떤 invariant를 표현하는가?
8. Idempotency Key Table에 Unique Constraint를 두는 이유는 무엇인가?
9. 같은 Idempotency Key가 다른 Request Body와 함께 오면 어떻게 처리해야 하는가?
10. 재고 차감에 단순 Upsert만 쓰기 어려운 이유는 무엇인가?
11. `UPDATE ... WHERE stock > 0` 같은 Atomic Conditional Update가 유용한 이유는 무엇인가?

## 면접 꼬리 질문

12. Unique Violation은 항상 시스템 장애인가?
13. NULL과 UNIQUE의 세부 의미를 DBMS마다 확인해야 하는 이유는 무엇인가?
14. Upsert가 해결하지 못하는 동시성 문제를 하나 설명하라.
15. 60초 안에 Unique Constraint와 Upsert를 설명하라.
