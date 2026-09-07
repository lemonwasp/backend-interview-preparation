# 14. Replication and Read Replicas — Knowledge Check

## 상태

- 답변: Pending
- Re-test: Pending

## 기본 질문

1. Replication의 목적 세 가지를 설명하라.
2. Synchronous와 Asynchronous Replication의 trade-off는 무엇인가?
3. Replication Lag란 무엇인가?
4. Read-after-write Consistency 문제가 어떻게 발생하는가?
5. Replica가 Backup을 대체하지 못하는 이유는 무엇인가?

## 설계 질문

6. 사용자가 Profile 수정 직후 이전 값이 보인다면 어떤 구조를 의심할 수 있는가?
7. Read-after-write가 필요한 요청을 어떻게 Routing할 수 있는가?
8. 어떤 종류의 Read가 Replica에 더 적합한가?
9. Primary Failover에서 Split-brain이 왜 위험한가?
10. Replica를 많이 추가해도 Write bottleneck이 남을 수 있는 이유는 무엇인가?
11. Application Instance 수와 DB Connection Pool 크기를 Replication 설계와 함께 봐야 하는 이유는 무엇인가?

## 면접 꼬리 질문

12. Async Replica가 최신 값을 보장하지 못한다면 왜 사용하는가?
13. Replica 승격 시 가장 먼저 확인해야 할 것은 무엇인가?
14. Replication과 Sharding의 목적 차이를 설명하라.
15. 60초 안에 Replication / Read Replica를 설명하라.
