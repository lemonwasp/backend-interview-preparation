# 12. Optimistic vs Pessimistic Locking — Knowledge Check

## 상태

- 답변: Pending
- Re-test: Pending

## 기본 질문

1. Optimistic Lock과 Pessimistic Lock의 핵심 차이는 무엇인가?
2. Version Column을 이용한 Optimistic Lock의 UPDATE 조건을 설명하라.
3. Optimistic Update 결과 Row Count가 0이면 무엇을 의미할 수 있는가?
4. `SELECT ... FOR UPDATE`는 어떤 방식에 해당하는가?
5. Pessimistic Lock의 대표 비용 세 가지를 말하라.

## 설계 질문

6. 게시글 수정 화면을 3분 동안 열어두는 서비스에는 어느 방식이 더 자연스러운가? 왜인가?
7. 남은 좌석 1개를 100명이 동시에 예약하면 어떤 전략을 고려하겠는가?
8. Hot Row에서 Optimistic Lock Retry가 폭증하는 이유는 무엇인가?
9. 외부 결제 API 호출을 Pessimistic Lock Transaction 내부에 넣으면 왜 위험한가?
10. ORM에서 Version Check를 Application SELECT 후 별도 UPDATE로 구현하면 왜 Race가 남는가?

## 면접 꼬리 질문

11. Optimistic Lock은 DB Lock을 전혀 사용하지 않는가?
12. Pessimistic Lock이 항상 더 안전한 설계인가?
13. Retry와 Idempotency는 Optimistic Lock과 어떤 관계가 있는가?
14. 충돌 빈도, Transaction 길이, Retry 비용을 기준으로 두 방식을 비교하라.
15. 60초 안에 Optimistic vs Pessimistic Locking을 설명하라.
