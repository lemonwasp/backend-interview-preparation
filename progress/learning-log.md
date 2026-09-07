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

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- 코드 또는 Linux/OS 도구 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Current Checkpoint

OS 문서는 19개, Networking 문서는 15개 주제까지 준비되었습니다. 대부분 Quiz와 Re-test가 Pending이므로 문서 생성 자체는 완료 증거가 아닙니다.

Networking에서 다음을 자료 없이 설명할 수 있어야 합니다.

1. HTTPS는 HTTP + TLS이며 TLS는 기밀성·무결성·인증을 제공한다.
2. TLS Handshake에서 인증과 키 합의를 한 뒤 Application Data는 주로 대칭키로 보호한다.
3. Certificate Chain은 Leaf → Intermediate → Trusted Root로 검증되고 Root 신뢰는 Trust Store에서 시작한다.
4. Forward Proxy는 Client를 대신하고 Reverse Proxy는 Server를 대신한다.
5. L4 Load Balancer와 L7 Load Balancer는 사용하는 계층 정보와 Routing 능력이 다르다.
6. Sticky Session은 편리하지만 Stateless Backend보다 확장성과 장애 대응에 제약이 있다.
7. HTTP Cache에서 Freshness와 Validation은 다른 개념이며 ETag를 이용하면 304 응답으로 재검증할 수 있다.
8. `no-cache`는 저장 금지가 아니라 재사용 전 검증을 요구하는 의미이고 `no-store`와 다르다.
9. 사용자별 응답을 Shared Cache할 때 Cache Key와 인증 경계를 잘못 설계하면 데이터 노출 위험이 있다.
10. Cache Stampede는 인기 Entry가 동시에 만료될 때 Origin으로 요청이 몰리는 문제다.

## Next Action

1. [TLS / HTTPS Quiz](../quizzes/networking/11-tls-https.md)
2. [Certificate / PKI Quiz](../quizzes/networking/12-certificate-pki.md)
3. [Forward / Reverse Proxy Quiz](../quizzes/networking/13-forward-reverse-proxy.md)
4. [Load Balancing Quiz](../quizzes/networking/14-load-balancing.md)
5. [CDN / HTTP Cache Quiz](../quizzes/networking/15-cdn-http-cache.md)
6. 다음 문서: WebSocket → SSE → gRPC → REST/RPC → API Gateway
