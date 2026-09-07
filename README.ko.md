# Backend Interview Preparation

[English](README.md) | **한국어**

백엔드 기술면접에 필요한 컴퓨터과학 기초를 체계적으로 익히고, 직접 설명할 수 있는 수준까지 끌어올리기 위한 저장소입니다.

## 학습 방법

1. 전문용어를 줄인 파인만식 설명을 읽는다.
2. 실제 백엔드 사례와 연결한다.
3. 자료를 보지 않고 자신의 말로 설명한다.
4. 확인 질문과 면접 꼬리 질문에 답한다.
5. 필요하면 코드나 운영체제 도구로 현상을 재현한다.
6. 틀린 답과 부족한 부분을 문서에 다시 반영한다.

## 학습 범위

| 영역 | 주요 주제 |
|---|---|
| 운영체제 | 프로세스, 스레드, 메모리, 동시성, I/O, Socket |
| 네트워크 | TCP/IP, UDP, DNS, HTTP, QUIC, TLS, Proxy, Load Balancing, 실시간 통신, RPC, API Gateway, Resilience, Connection 관리 |
| 데이터베이스 | 관계형 모델, Index, Query 실행, Transaction, Isolation, MVCC, Lock, Normalization, Constraint, Replication, Sharding, Connection Pool, ORM, Migration, Pagination, Cache |
| 런타임·동시성 | Thread Pool, 비동기 I/O, GC, 동기화 |
| 시스템 설계 | Cache, Queue, Replication, Sharding, Reliability |

## 완료 기준

- 전문용어 없이 쉽게 설명할 수 있다.
- 왜 필요한지 설명할 수 있다.
- 내부 동작의 인과관계를 설명할 수 있다.
- 장점과 비용을 함께 말할 수 있다.
- 백엔드 개발 사례와 연결할 수 있다.
- 자료 없이 꼬리 질문에 답할 수 있다.
- 필요한 경우 코드나 운영체제 도구로 현상을 재현할 수 있다.

## 진행 상황

| 영역 | 현재 주제 | 상태 |
|---|---|---|
| 운영체제 | 19개 문서 준비, Quiz/Re-test 대기 | 복습 |
| 네트워크 | 30개 주제 준비, Mock Interview 대기 | 복습 |
| 데이터베이스 | Cache Consistency | 학습 중 |
| 런타임·동시성 | 시작 전 | 대기 |
| 시스템 설계 | 시작 전 | 대기 |

## 현재 학습

- [20. Cache Consistency](docs/databases/20-cache-consistency.md)
- [20. 이해도 확인 문제](quizzes/databases/20-cache-consistency.md)

## 데이터베이스 학습 목록

1. [관계형 모델과 Key](docs/databases/01-relational-model-and-keys.md) · [확인 문제](quizzes/databases/01-relational-model-and-keys.md)
2. [B-Tree Index](docs/databases/02-b-tree-index.md) · [확인 문제](quizzes/databases/02-b-tree-index.md)
3. [Clustered / Non-clustered Index](docs/databases/03-clustered-vs-nonclustered-index.md) · [확인 문제](quizzes/databases/03-clustered-vs-nonclustered-index.md)
4. [Query Execution / EXPLAIN](docs/databases/04-query-execution-and-explain.md) · [확인 문제](quizzes/databases/04-query-execution-and-explain.md)
5. [Transaction / ACID](docs/databases/05-transaction-and-acid.md) · [확인 문제](quizzes/databases/05-transaction-and-acid.md)
6. [Transaction Isolation Level](docs/databases/06-isolation-levels.md) · [확인 문제](quizzes/databases/06-isolation-levels.md)
7. [동시성 이상 현상](docs/databases/07-concurrency-anomalies.md) · [확인 문제](quizzes/databases/07-concurrency-anomalies.md)
8. [MVCC](docs/databases/08-mvcc.md) · [확인 문제](quizzes/databases/08-mvcc.md)
9. [Database Lock](docs/databases/09-database-locks.md) · [확인 문제](quizzes/databases/09-database-locks.md)
10. [Database Deadlock](docs/databases/10-deadlocks.md) · [확인 문제](quizzes/databases/10-deadlocks.md)
11. [Normalization](docs/databases/11-normalization.md) · [확인 문제](quizzes/databases/11-normalization.md)
12. [Optimistic / Pessimistic Locking](docs/databases/12-optimistic-vs-pessimistic-locking.md) · [확인 문제](quizzes/databases/12-optimistic-vs-pessimistic-locking.md)
13. [Unique Constraint / Upsert](docs/databases/13-unique-constraint-and-upsert.md) · [확인 문제](quizzes/databases/13-unique-constraint-and-upsert.md)
14. [Replication / Read Replica](docs/databases/14-replication-and-read-replicas.md) · [확인 문제](quizzes/databases/14-replication-and-read-replicas.md)
15. [Partitioning / Sharding](docs/databases/15-partitioning-and-sharding.md) · [확인 문제](quizzes/databases/15-partitioning-and-sharding.md)
16. [DB Connection Pool / Transaction Boundary](docs/databases/16-db-connection-pool-transaction-boundary.md) · [확인 문제](quizzes/databases/16-db-connection-pool-transaction-boundary.md)
17. [ORM / N+1](docs/databases/17-orm-n-plus-one.md) · [확인 문제](quizzes/databases/17-orm-n-plus-one.md)
18. [Schema Migration](docs/databases/18-schema-migration.md) · [확인 문제](quizzes/databases/18-schema-migration.md)
19. [Pagination / Large Data Access](docs/databases/19-pagination-large-data-access.md) · [확인 문제](quizzes/databases/19-pagination-large-data-access.md)
20. [Cache Consistency](docs/databases/20-cache-consistency.md) · [확인 문제](quizzes/databases/20-cache-consistency.md)

## 네트워크 트랙

Networking은 29개 개념 문서 + 1개 60초 답변 총정리 + 40문항 Mock Interview까지 준비되어 있습니다. Mock Interview와 재시험을 통과하기 전까지 Completed로 처리하지 않습니다.

- [Networking 총정리](docs/networking/30-networking-review-60-second-answers.md)
- [Networking Mock Interview](quizzes/networking/30-networking-review-60-second-answers.md)

## 운영체제 트랙

OS는 운영체제·커널 기초부터 File Descriptor, DMA, Zero-copy, Socket Internals, OS 내부 TCP 처리까지 19개 주제가 준비되어 있습니다. 대부분 Quiz와 Re-test가 남아 있습니다.

- [OS 첫 주제](docs/operating-systems/01-os-and-kernel.md)
- [OS 19번](docs/operating-systems/19-tcp-in-the-os.md)
- [학습 기록](progress/learning-log.md)

## 다음 데이터베이스 주제

Backup / PITR → WAL / Checkpoint / Crash Recovery → DB 장애 시나리오 → Outbox / CDC → Database 총정리 순으로 진행한 뒤 Runtime & Concurrency로 넘어갑니다.

- [전체 로드맵](ROADMAP.md)

## 언어 운영

정확한 이해를 위해 상세 학습 문서는 한국어로 먼저 작성합니다. 개념을 충분히 이해한 뒤에는 기술면접에 사용할 수 있는 짧은 영어 답변도 추가합니다.
