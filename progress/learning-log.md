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

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- 코드 또는 Linux/OS 도구 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Current Checkpoint

OS 문서는 19개, Networking 문서는 25개 주제까지 준비되었습니다. 대부분 Quiz와 Re-test가 Pending이므로 문서 생성 자체는 완료 증거가 아닙니다.

이번 Resilience 묶음에서는 다음을 자료 없이 설명할 수 있어야 합니다.

1. Rate Limiting은 유입량을 제한하고 Token Bucket은 평균 속도를 유지하면서 제한된 Burst를 허용한다.
2. Timeout은 개별 호출을 무한정 기다리지 않게 하며 downstream timeout은 전체 요청 budget보다 작아야 한다.
3. Retry는 transient failure 복구에 유용하지만 여러 계층이 독립적으로 수행하면 Retry Amplification이 발생할 수 있다.
4. Exponential Backoff는 재시도 간격을 늘리고 Jitter는 여러 Client의 동시 재시도를 분산한다.
5. Retry 전에 요청의 Idempotency를 확인해야 하며 필요하면 Idempotency Key를 사용한다.
6. Circuit Breaker는 Closed / Open / Half-Open 상태로 반복 장애를 fail fast하여 격리한다.
7. Timeout은 개별 호출 대기 제한이고 Circuit Breaker는 반복 실패 시 호출 자체를 잠시 막는 패턴이다.
8. Bulkhead는 Thread Pool, Connection Pool, Semaphore, Queue 같은 자원을 분리해 하나의 장애가 전체 시스템을 고갈시키는 것을 막는다.
9. 무한 Queue는 과부하를 해결하지 않고 latency와 memory 문제를 키울 수 있다.
10. Resilience는 장애가 절대 발생하지 않는 상태가 아니라 장애의 피해 범위를 제한하고 핵심 기능을 유지하는 능력이다.

## Next Action

1. [Rate Limiting Quiz](../quizzes/networking/21-rate-limiting.md)
2. [Timeout Budget Quiz](../quizzes/networking/22-timeout-budget.md)
3. [Retry / Backoff / Jitter Quiz](../quizzes/networking/23-retry-backoff-jitter.md)
4. [Circuit Breaker Quiz](../quizzes/networking/24-circuit-breaker.md)
5. [Bulkhead / Resilience Quiz](../quizzes/networking/25-bulkhead-resilience-patterns.md)
6. 다음 문서: Idempotency Key → Backpressure 심화 → Connection Pooling → NAT / Ephemeral Port
7. 그 다음에는 Networking 전체를 60초 답변 중심으로 압축 복습한 뒤 Database 트랙으로 이동
