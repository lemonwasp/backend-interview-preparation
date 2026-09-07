# 11. Normalization — Knowledge Check

## 상태

- 답변: Pending
- Re-test: Pending

## 기본 질문

1. Normalization이 필요한 가장 큰 이유는 무엇인가?
2. Update / Insert / Delete Anomaly를 각각 설명하라.
3. Functional Dependency란 무엇인가?
4. 1NF의 핵심을 설명하라.
5. Composite Key와 관련해 2NF가 해결하려는 문제는 무엇인가?
6. 3NF가 줄이려는 Transitive Dependency란 무엇인가?
7. BCNF를 3NF보다 더 엄격한 규칙이라고 하는 이유는 무엇인가?

## 설계 질문

8. `orders(order_id, customer_id, customer_name, customer_address)`가 반복될 때 어떤 문제가 생기는가?
9. 주문 당시 가격을 `order_items.price_at_purchase`에 저장하는 것은 왜 반드시 나쁜 중복이 아닌가?
10. Normalization과 Denormalization의 trade-off를 설명하라.
11. OLTP와 분석/Read Model에서 정규화 전략이 달라질 수 있는 이유는 무엇인가?
12. Denormalization을 도입한다면 어떤 동기화 위험을 관리해야 하는가?

## 면접 꼬리 질문

13. "정규화는 무조건 3NF까지 해야 한다"는 주장에 어떻게 답하겠는가?
14. Join이 많아져 느려졌다면 가장 먼저 무조건 Denormalization해야 하는가?
15. 60초 안에 Normalization을 설명하라.
