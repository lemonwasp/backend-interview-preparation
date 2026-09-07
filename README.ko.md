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
| 데이터베이스 | 25개 주제 준비, Mock Interview 대기 | 복습 |
| 런타임·동시성 | Runtime Observability | 학습 중 |
| 시스템 설계 | 시작 전 | 대기 |

## 현재 학습

- [15. Runtime Observability](docs/runtime-concurrency/15-runtime-observability.md)
- [15. 이해도 확인 문제](quizzes/runtime-concurrency/15-runtime-observability.md)

## 런타임·동시성 학습 목록

1. [Runtime Process Model](docs/runtime-concurrency/01-runtime-process-model.md) · [확인 문제](quizzes/runtime-concurrency/01-runtime-process-model.md)
2. [Thread Pool](docs/runtime-concurrency/02-thread-pool.md) · [확인 문제](quizzes/runtime-concurrency/02-thread-pool.md)
3. [Task / Future / Promise](docs/runtime-concurrency/03-task-future-promise.md) · [확인 문제](quizzes/runtime-concurrency/03-task-future-promise.md)
4. [async / await 내부 동작](docs/runtime-concurrency/04-async-await-internals.md) · [확인 문제](quizzes/runtime-concurrency/04-async-await-internals.md)
5. [Synchronization Primitives](docs/runtime-concurrency/05-synchronization-primitives.md) · [확인 문제](quizzes/runtime-concurrency/05-synchronization-primitives.md)
6. [Concurrent Collections](docs/runtime-concurrency/06-concurrent-collections.md) · [확인 문제](quizzes/runtime-concurrency/06-concurrent-collections.md)
7. [Cancellation / Timeout](docs/runtime-concurrency/07-cancellation-and-timeout.md) · [확인 문제](quizzes/runtime-concurrency/07-cancellation-and-timeout.md)
8. [GC Generations](docs/runtime-concurrency/08-gc-generations.md) · [확인 문제](quizzes/runtime-concurrency/08-gc-generations.md)
9. [Allocation / Boxing](docs/runtime-concurrency/09-allocation-and-boxing.md) · [확인 문제](quizzes/runtime-concurrency/09-allocation-and-boxing.md)
10. [Managed Memory Leak](docs/runtime-concurrency/10-managed-memory-leak.md) · [확인 문제](quizzes/runtime-concurrency/10-managed-memory-leak.md)
11. [IDisposable / Resource Lifetime](docs/runtime-concurrency/11-idisposable-resource-lifetime.md) · [확인 문제](quizzes/runtime-concurrency/11-idisposable-resource-lifetime.md)
12. [Async Stream / Channel](docs/runtime-concurrency/12-async-streams-and-channels.md) · [확인 문제](quizzes/runtime-concurrency/12-async-streams-and-channels.md)
13. [ThreadPool Starvation 진단](docs/runtime-concurrency/13-threadpool-starvation-diagnostics.md) · [확인 문제](quizzes/runtime-concurrency/13-threadpool-starvation-diagnostics.md)
14. [ExecutionContext / Context Flow](docs/runtime-concurrency/14-executioncontext-context-flow.md) · [확인 문제](quizzes/runtime-concurrency/14-executioncontext-context-flow.md)
15. [Runtime Observability](docs/runtime-concurrency/15-runtime-observability.md) · [확인 문제](quizzes/runtime-concurrency/15-runtime-observability.md)

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

## 다음 런타임 주제

Runtime & Concurrency 총정리와 Mock Interview를 만든 뒤 신규 주제 준비는 System Design으로 이동합니다. 기존 OS / Networking / Database / Runtime은 Quiz와 Re-test를 통과하기 전까지 Prepared 상태를 유지합니다.

- [전체 로드맵](ROADMAP.md)

## 언어 운영

정확한 이해를 위해 상세 학습 문서는 한국어로 먼저 작성합니다. 개념을 충분히 이해한 뒤에는 기술면접에 사용할 수 있는 짧은 영어 답변도 추가합니다.
