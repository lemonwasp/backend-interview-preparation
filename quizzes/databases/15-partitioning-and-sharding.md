# 15. Partitioning and Sharding — Knowledge Check

## 상태

- 답변: Pending
- Re-test: Pending

## 기본 질문

1. Partitioning과 Sharding의 핵심 차이는 무엇인가?
2. Range / List / Hash Partitioning을 각각 설명하라.
3. Partition Pruning이란 무엇인가?
4. 좋은 Shard Key의 조건은 무엇인가?
5. Hotspot Shard는 어떻게 발생할 수 있는가?

## 설계 질문

6. 월별 주문 데이터를 Range Partitioning하는 이유는 무엇인가?
7. `hash(user_id) % N` 방식에서 Shard 수 변경이 어려운 이유는 무엇인가?
8. Consistent Hashing이 해결하려는 문제는 무엇인가?
9. Cross-shard Join / Aggregate / Transaction이 어려운 이유는 무엇인가?
10. Global Unique ID를 Sharded DB에서 어떻게 만들 수 있는가?
11. Rebalancing 시 어떤 운영 문제가 생기는가?
12. Replication과 Sharding을 함께 사용하는 구조를 설명하라.

## 면접 꼬리 질문

13. 데이터가 많아졌다고 바로 Sharding하면 안 되는 이유는 무엇인가?
14. Sharding 전에 검토할 수 있는 대안 네 가지를 말하라.
15. 60초 안에 Partitioning vs Sharding을 설명하라.
