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
| 데이터베이스 | 관계형 모델, Index, Query 실행, Transaction, Isolation, MVCC, Lock, Normalization, Constraint, Replication, Sharding, Connection Pool, ORM, Migration, Pagination, Cache, Backup, Recovery, CDC |
| 런타임·동시성 | CLR/Runtime Model, Thread Pool, Task, async/await, Synchronization, Concurrent Collection, Cancellation, GC, Allocation, Memory/Resource Lifetime, Async Stream, Context Flow, Observability |
| 시스템 설계 | 요구사항, Capacity Estimation, Stateless Service, Cache, Queue, Load Balancing, DB Scaling, Consistency, Idempotency, Rate Limiting, Distributed Lock, Service Boundary, SLO, Graceful Degradation, Failure Mode, Multi-region, DR |

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
| 데이터베이스 | 25개 주제 준비, Mock Interview 대기 | 복습 |
| 런타임·동시성 | 16개 주제 준비, Mock Interview 대기 | 복습 |
| 시스템 설계 | 16개 주제 준비, Mock Interview 대기 | 복습 |

## 현재 학습

- [16. System Design 총정리 — 60초 답변 / Trade-off](docs/system-design/16-system-design-review-60-second-answers.md)
- [System Design Mock Interview](quizzes/system-design/16-system-design-review-60-second-answers.md)

## 시스템 설계 학습 목록

1. [Requirements / Capacity Estimation](docs/system-design/01-requirements-and-capacity-estimation.md) · [확인 문제](quizzes/system-design/01-requirements-and-capacity-estimation.md)
2. [Stateless Service](docs/system-design/02-stateless-service.md) · [확인 문제](quizzes/system-design/02-stateless-service.md)
3. [Cache Design](docs/system-design/03-cache-design.md) · [확인 문제](quizzes/system-design/03-cache-design.md)
4. [Message Queue](docs/system-design/04-message-queue.md) · [확인 문제](quizzes/system-design/04-message-queue.md)
5. [Load Balancing / Horizontal Scaling](docs/system-design/05-load-balancing-horizontal-scaling.md) · [확인 문제](quizzes/system-design/05-load-balancing-horizontal-scaling.md)
6. [Database Scaling / Read-Write Pattern](docs/system-design/06-database-scaling-read-write-patterns.md) · [확인 문제](quizzes/system-design/06-database-scaling-read-write-patterns.md)
7. [Consistency / Availability](docs/system-design/07-consistency-and-availability.md) · [확인 문제](quizzes/system-design/07-consistency-and-availability.md)
8. [Distributed Idempotency](docs/system-design/08-distributed-idempotency.md) · [확인 문제](quizzes/system-design/08-distributed-idempotency.md)
9. [Rate Limiting](docs/system-design/09-rate-limiting.md) · [확인 문제](quizzes/system-design/09-rate-limiting.md)
10. [Distributed Lock](docs/system-design/10-distributed-lock.md) · [확인 문제](quizzes/system-design/10-distributed-lock.md)
11. [Service Boundary / Ownership](docs/system-design/11-service-boundary-and-ownership.md) · [확인 문제](quizzes/system-design/11-service-boundary-and-ownership.md)
12. [Observability / SLO](docs/system-design/12-observability-and-slo.md) · [확인 문제](quizzes/system-design/12-observability-and-slo.md)
13. [Graceful Degradation](docs/system-design/13-graceful-degradation.md) · [확인 문제](quizzes/system-design/13-graceful-degradation.md)
14. [Failure Mode Reasoning](docs/system-design/14-failure-mode-reasoning.md) · [확인 문제](quizzes/system-design/14-failure-mode-reasoning.md)
15. [Multi-region / Disaster Recovery](docs/system-design/15-multi-region-and-disaster-recovery.md) · [확인 문제](quizzes/system-design/15-multi-region-and-disaster-recovery.md)
16. [System Design 총정리 — 60초 답변 / Trade-off](docs/system-design/16-system-design-review-60-second-answers.md) · [Mock Interview](quizzes/system-design/16-system-design-review-60-second-answers.md)

## 런타임·동시성 트랙

Runtime & Concurrency는 15개 개념 문서 + 1개 60초 답변 총정리 + 40문항 Mock Interview까지 준비되어 있습니다. Mock Interview와 재시험을 통과하기 전까지 Completed로 처리하지 않습니다.

- [Runtime & Concurrency 총정리](docs/runtime-concurrency/16-runtime-concurrency-review-60-second-answers.md)
- [Runtime & Concurrency Mock Interview](quizzes/runtime-concurrency/16-runtime-concurrency-review-60-second-answers.md)

## 데이터베이스 트랙

Database는 24개 개념 문서 + 1개 60초 답변 총정리 + 40문항 Mock Interview까지 준비되어 있습니다. Mock Interview와 재시험을 통과하기 전까지 Completed로 처리하지 않습니다.

- [Database 총정리](docs/databases/25-database-review-60-second-answers.md)
- [Database Mock Interview](quizzes/databases/25-database-review-60-second-answers.md)

## 네트워크 트랙

Networking은 29개 개념 문서 + 1개 60초 답변 총정리 + 40문항 Mock Interview까지 준비되어 있습니다. Mock Interview와 재시험을 통과하기 전까지 Completed로 처리하지 않습니다.

- [Networking 총정리](docs/networking/30-networking-review-60-second-answers.md)
- [Networking Mock Interview](quizzes/networking/30-networking-review-60-second-answers.md)

## 운영체제 트랙

OS는 운영체제·커널 기초부터 File Descriptor, DMA, Zero-copy, Socket Internals, OS 내부 TCP 처리까지 19개 주제가 준비되어 있습니다. 대부분 Quiz와 Re-test가 남아 있습니다.

- [OS 첫 주제](docs/operating-systems/01-os-and-kernel.md)
- [OS 19번](docs/operating-systems/19-tcp-in-the-os.md)
- [학습 기록](progress/learning-log.md)

## 복습 단계

신규 개념 추가는 일단 중단합니다. 이제 Quiz, Mock Interview, 자료 없이 설명, D+1/D+7 재시험을 통해 Prepared를 실제 Completed로 전환합니다. 문서가 존재한다는 이유만으로 완료 처리하지 않습니다.

- [전체 로드맵](ROADMAP.md)

## 언어 운영

정확한 이해를 위해 상세 학습 문서는 한국어로 먼저 작성합니다. 개념을 충분히 이해한 뒤에는 기술면접에 사용할 수 있는 짧은 영어 답변도 추가합니다.
