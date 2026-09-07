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

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- 코드 또는 Linux/OS 도구 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Current Checkpoint

OS 문서는 19개, Networking은 29개 개념 문서 + 1개 총정리 문서까지 준비되었습니다. 대부분 Quiz와 Re-test가 Pending이므로 문서 생성 자체는 완료 증거가 아닙니다.

Networking 마무리 묶음에서는 다음을 자료 없이 설명할 수 있어야 합니다.

1. Idempotency Key는 Timeout/Retry로 같은 비즈니스 요청이 중복 도착해도 중복 부작용을 막기 위한 요청 식별자다.
2. 같은 Idempotency Key를 가진 동시 요청은 Unique Constraint나 Atomic Insert 등으로 최초 처리자를 하나로 정해야 한다.
3. Backpressure는 Producer가 Consumer capacity를 초과할 때 bounded queue, concurrency limit, load shedding 등으로 upstream 속도를 조절하는 메커니즘이다.
4. Buffer는 burst는 흡수하지만 평균 입력 속도가 평균 처리 속도보다 계속 높으면 근본 해결책이 아니다.
5. Connection Pool은 handshake 비용을 줄이는 성능 최적화이면서 downstream concurrency를 제한하는 보호 장치이기도 하다.
6. Pool이 너무 크면 downstream contention을 키울 수 있으므로 전체 App Instance 수와 downstream capacity를 같이 봐야 한다.
7. Client outbound TCP connection은 ephemeral source port를 사용하고 NAT는 private tuple을 public tuple로 매핑한다.
8. Connection churn이 크면 TIME_WAIT, ephemeral port, NAT mapping pressure가 증가할 수 있으므로 keep-alive와 pooling이 중요하다.
9. Networking 면접에서는 개별 용어 암기보다 TCP/HTTP/TLS/Proxy/Resilience/Connection Resource를 하나의 요청 흐름으로 연결해서 설명할 수 있어야 한다.

## Networking Final Review

- [Networking 60초 답변 총정리](../docs/networking/30-networking-review-60-second-answers.md)
- [40문항 Networking Mock Interview](../quizzes/networking/30-networking-review-60-second-answers.md)

### Networking을 Completed로 올리는 최소 기준

1. Mock Interview 40문항 중 최소 32개 이상을 핵심 개념 혼동 없이 답변
2. TCP vs UDP, HTTP/1.1/2/3, Proxy, Load Balancer, REST/gRPC, Timeout/Circuit Breaker, Rate Limit/Backpressure 비교 질문 통과
3. 5개 장애 시나리오에서 관찰 지표 → 원인 가설 → 보호 조치 순서로 답변
4. 틀린 항목을 해당 문서에 반영
5. 최소 D+1 재시험 수행

## Next Action

1. 새 Networking 개념 추가는 일단 중단
2. [Networking Mock Interview](../quizzes/networking/30-networking-review-60-second-answers.md) 진행
3. OS Quiz도 병행해 Prepared를 실제 Completed로 전환
4. 다음 신규 학습 트랙은 Database Fundamentals
