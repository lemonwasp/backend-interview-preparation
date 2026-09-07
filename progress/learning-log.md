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

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- 코드 또는 Linux/OS 도구 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Current Checkpoint

OS 문서는 19개, Networking 문서는 10개 주제까지 준비되었습니다. 대부분 Quiz와 Re-test가 Pending이므로 문서 생성 자체는 완료 증거가 아닙니다.

Networking에서 다음을 자료 없이 설명할 수 있어야 합니다.

1. TCP는 reliable ordered byte stream이고 UDP는 datagram 기반이라는 차이
2. DNS Resolver / Root / TLD / Authoritative Server와 TTL Cache의 역할
3. HTTP/1.1 Keep-Alive가 connection churn과 handshake 비용을 줄이는 이유
4. HTTP/1.1 pipelining의 application-level HOL Blocking
5. HTTP/2 multiplexing이 application-level HOL을 줄이지만 TCP-level HOL은 남는 이유
6. HTTP/3가 QUIC을 사용해 stream별 loss 영향 범위를 줄이는 방식
7. QUIC이 UDP 기반이지만 reliability, congestion control, TLS 1.3을 자체 제공한다는 점
8. HTTP/3가 모든 환경에서 무조건 더 빠른 것은 아니라는 점

## Next Action

1. [UDP vs TCP Quiz](../quizzes/networking/06-udp-vs-tcp.md)
2. [DNS Quiz](../quizzes/networking/07-dns.md)
3. [HTTP/1.1 Quiz](../quizzes/networking/08-http-1-1.md)
4. [HTTP/2 Quiz](../quizzes/networking/09-http-2.md)
5. [HTTP/3 & QUIC Quiz](../quizzes/networking/10-http-3-quic.md)
6. 다음 문서: TLS/HTTPS → Certificate/PKI → Proxy/Load Balancing
