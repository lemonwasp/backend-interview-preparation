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

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- 코드 또는 Linux/OS 도구 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Current Checkpoint

OS 문서는 19개, Networking 문서는 20개 주제까지 준비되었습니다. 대부분 Quiz와 Re-test가 Pending이므로 문서 생성 자체는 완료 증거가 아닙니다.

Networking에서 다음을 자료 없이 설명할 수 있어야 합니다.

1. WebSocket은 장기 연결 위에서 양방향 통신을 제공하고 scale-out 시 연결 위치와 cross-node message delivery를 고려해야 한다.
2. SSE는 HTTP 기반 Server → Client 단방향 stream이며 자동 재연결이 exactly-once delivery를 의미하지 않는다.
3. gRPC는 Protobuf 자체가 아니라 RPC framework이며 HTTP/2, code generation, streaming, deadline을 활용한다.
4. REST와 RPC의 핵심 차이는 JSON vs Binary가 아니라 Resource-oriented vs Action/Method-oriented 추상화다.
5. Public API에는 REST, 내부 service-to-service에는 gRPC를 함께 사용하는 구조가 자연스러울 수 있다.
6. API Gateway는 Routing, Authentication, Rate Limiting 등 공통 정책을 적용하지만 domain authorization까지 모두 대신하는 것은 아니다.
7. Gateway도 bottleneck/SPOF가 될 수 있으므로 horizontal scaling, health check와 observability가 필요하다.
8. Client/Gateway/Service가 각각 독립적으로 Retry하면 Retry Amplification이 발생할 수 있다.

## Next Action

1. [WebSocket Quiz](../quizzes/networking/16-websocket.md)
2. [SSE Quiz](../quizzes/networking/17-server-sent-events.md)
3. [gRPC Quiz](../quizzes/networking/18-grpc.md)
4. [REST vs RPC Quiz](../quizzes/networking/19-rest-vs-rpc.md)
5. [API Gateway Quiz](../quizzes/networking/20-api-gateway.md)
6. 다음 문서: Rate Limiting → Timeout Budget → Retry / Backoff → Circuit Breaker
