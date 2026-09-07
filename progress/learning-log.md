# Learning Log

학습 완료 여부는 문서를 읽은 시점이 아니라, 확인 질문과 재시험 결과로 판단합니다.

| Date | Area | Topic | Explanation | Quiz | Re-test | Status |
|---|---|---|---|---|---|---|
| 2026-09-05 | OS | OS and Kernel | Completed | Pending | Pending | Learning |
| 2026-09-07 | OS | User/Kernel Mode & System Calls | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | Process vs Thread | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | Process Address Space | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | Context Switching | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | CPU Scheduling | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | Blocking / Non-blocking / Sync / Async | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | Concurrency vs Parallelism | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | Race Condition / Lock / Deadlock | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Memory Visibility / Memory Barrier | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Virtual Memory Deep Dive | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Paging / Page Fault | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Page Cache / File I/O | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | I/O Multiplexing | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | File Descriptor | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Interrupt / DMA | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Zero-copy / sendfile | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Socket Internals | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | TCP Processing Inside the OS | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | TCP/IP Layers | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | TCP 3-way Handshake | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | TCP Termination / TIME_WAIT | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | TCP Retransmission / RTO | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Flow Control / Congestion Control | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | UDP vs TCP | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | DNS | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | HTTP/1.1 | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | HTTP/2 | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | HTTP/3 & QUIC | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | TLS / HTTPS | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Certificate / PKI | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Forward / Reverse Proxy | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Load Balancing | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | CDN / HTTP Cache | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | WebSocket | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Server-Sent Events | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | gRPC | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | REST vs RPC | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | API Gateway | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Rate Limiting | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Timeout Budget | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Retry / Backoff / Jitter | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Circuit Breaker | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Bulkhead / Resilience Patterns | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Idempotency Key | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Backpressure Deep Dive | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Connection Pooling | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | NAT / Ephemeral Ports | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Networking Review / 60-second Answers | Prepared | Pending | Pending | Review |
| 2026-09-08 | Database | Relational Model / Keys | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Database | B-Tree Index | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Database | Clustered / Non-clustered Index | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Database | Query Execution / EXPLAIN | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Database | Transaction / ACID | Prepared | Pending | Pending | Learning |

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- 코드 또는 Linux/OS/DB 도구 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Current Checkpoint

OS는 19개, Networking은 29개 개념 + 1개 총정리, Database는 첫 5개 주제까지 준비되었습니다. 문서 생성 자체는 학습 완료 증거가 아닙니다.

Database 첫 묶음에서는 다음을 자료 없이 설명할 수 있어야 합니다.

1. Primary Key는 Row 식별 Constraint이고 Index는 검색용 Access Structure이므로 같은 개념이 아니다.
2. Foreign Key는 다른 Table의 Key를 참조해 Referential Integrity를 유지한다.
3. B-Tree 계열 Index는 높은 Branching Factor로 Tree 높이를 낮추고 Equality와 Range Search를 지원한다.
4. Composite Index는 Column 순서가 중요하고 Index가 존재해도 Selectivity와 Cost에 따라 Optimizer가 사용하지 않을 수 있다.
5. Clustered Index와 Non-clustered Index의 핵심 차이는 실제 Row 저장 구조와의 관계이며 DBMS별 구현 차이를 구분해야 한다.
6. EXPLAIN에서는 Index 사용 여부뿐 아니라 읽은 Row 수, Join 방식, Sort/Hash Spill, Estimated vs Actual Rows를 본다.
7. Full Table Scan은 많은 Row가 필요한 경우 Index Random Lookup보다 합리적일 수 있다.
8. Transaction은 여러 DB 작업을 하나의 논리적 단위로 묶고 ACID는 Atomicity / Consistency / Isolation / Durability를 설명한다.
9. WAL은 Data Page보다 복구용 Log를 먼저 안전하게 기록하는 원칙이다.
10. Local DB Transaction으로 외부 API까지 자동으로 Atomic하게 Rollback할 수는 없다.

## Next Action

1. [Relational Model / Keys Quiz](../quizzes/databases/01-relational-model-and-keys.md)
2. [B-Tree Index Quiz](../quizzes/databases/02-b-tree-index.md)
3. [Clustered / Non-clustered Index Quiz](../quizzes/databases/03-clustered-vs-nonclustered-index.md)
4. [Query Execution / EXPLAIN Quiz](../quizzes/databases/04-query-execution-and-explain.md)
5. [Transaction / ACID Quiz](../quizzes/databases/05-transaction-and-acid.md)
6. 다음 문서: Isolation Level → concurrency anomalies → MVCC → Lock / Deadlock
7. Networking Mock Interview와 OS Quiz는 별도 복습 세션에서 Prepared를 Completed로 전환
