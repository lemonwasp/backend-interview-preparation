# Backend Interview Preparation

[English](README.md) | **한국어**

백엔드 기술면접에 필요한 컴퓨터과학 기초를 체계적으로 익히고,
직접 설명할 수 있는 수준까지 끌어올리기 위한 저장소입니다.

## 학습 방법

각 개념은 다음 순서로 공부합니다.

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
| 네트워크 | TCP/IP, UDP, DNS, HTTP, QUIC, TLS, Proxy, Load Balancing, CDN, Cache, WebSocket, SSE, gRPC, API Gateway, Resilience |
| 데이터베이스 | Index, Transaction, Isolation, Lock, Query |
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
| 네트워크 | Bulkhead / Resilience Patterns | 학습 중 |
| 데이터베이스 | 시작 전 | 대기 |
| 런타임·동시성 | 시작 전 | 대기 |
| 시스템 설계 | 시작 전 | 대기 |

## 현재 학습

- [25. Bulkhead / Resilience Patterns](docs/networking/25-bulkhead-resilience-patterns.md)
- [25. 이해도 확인 문제](quizzes/networking/25-bulkhead-resilience-patterns.md)

## 네트워크 학습 목록

1. [TCP/IP 계층](docs/networking/01-tcp-ip-layers.md) · [확인 문제](quizzes/networking/01-tcp-ip-layers.md)
2. [TCP 3-way Handshake](docs/networking/02-tcp-three-way-handshake.md) · [확인 문제](quizzes/networking/02-tcp-three-way-handshake.md)
3. [TCP 종료와 TIME_WAIT](docs/networking/03-tcp-termination-time-wait.md) · [확인 문제](quizzes/networking/03-tcp-termination-time-wait.md)
4. [TCP Retransmission / RTO](docs/networking/04-tcp-retransmission-rto.md) · [확인 문제](quizzes/networking/04-tcp-retransmission-rto.md)
5. [TCP Flow Control / Congestion Control](docs/networking/05-flow-congestion-control.md) · [확인 문제](quizzes/networking/05-flow-congestion-control.md)
6. [UDP vs TCP](docs/networking/06-udp-vs-tcp.md) · [확인 문제](quizzes/networking/06-udp-vs-tcp.md)
7. [DNS](docs/networking/07-dns.md) · [확인 문제](quizzes/networking/07-dns.md)
8. [HTTP/1.1](docs/networking/08-http-1-1.md) · [확인 문제](quizzes/networking/08-http-1-1.md)
9. [HTTP/2](docs/networking/09-http-2.md) · [확인 문제](quizzes/networking/09-http-2.md)
10. [HTTP/3 & QUIC](docs/networking/10-http-3-quic.md) · [확인 문제](quizzes/networking/10-http-3-quic.md)
11. [TLS / HTTPS](docs/networking/11-tls-https.md) · [확인 문제](quizzes/networking/11-tls-https.md)
12. [Certificate / PKI](docs/networking/12-certificate-pki.md) · [확인 문제](quizzes/networking/12-certificate-pki.md)
13. [Forward Proxy / Reverse Proxy](docs/networking/13-forward-reverse-proxy.md) · [확인 문제](quizzes/networking/13-forward-reverse-proxy.md)
14. [Load Balancing](docs/networking/14-load-balancing.md) · [확인 문제](quizzes/networking/14-load-balancing.md)
15. [CDN / HTTP Cache](docs/networking/15-cdn-http-cache.md) · [확인 문제](quizzes/networking/15-cdn-http-cache.md)
16. [WebSocket](docs/networking/16-websocket.md) · [확인 문제](quizzes/networking/16-websocket.md)
17. [Server-Sent Events](docs/networking/17-server-sent-events.md) · [확인 문제](quizzes/networking/17-server-sent-events.md)
18. [gRPC](docs/networking/18-grpc.md) · [확인 문제](quizzes/networking/18-grpc.md)
19. [REST vs RPC](docs/networking/19-rest-vs-rpc.md) · [확인 문제](quizzes/networking/19-rest-vs-rpc.md)
20. [API Gateway](docs/networking/20-api-gateway.md) · [확인 문제](quizzes/networking/20-api-gateway.md)
21. [Rate Limiting](docs/networking/21-rate-limiting.md) · [확인 문제](quizzes/networking/21-rate-limiting.md)
22. [Timeout Budget](docs/networking/22-timeout-budget.md) · [확인 문제](quizzes/networking/22-timeout-budget.md)
23. [Retry / Exponential Backoff / Jitter](docs/networking/23-retry-backoff-jitter.md) · [확인 문제](quizzes/networking/23-retry-backoff-jitter.md)
24. [Circuit Breaker](docs/networking/24-circuit-breaker.md) · [확인 문제](quizzes/networking/24-circuit-breaker.md)
25. [Bulkhead / Resilience Patterns](docs/networking/25-bulkhead-resilience-patterns.md) · [확인 문제](quizzes/networking/25-bulkhead-resilience-patterns.md)

## 운영체제 트랙

OS는 운영체제·커널 기초부터 File Descriptor, DMA, Zero-copy, Socket Internals, OS 내부 TCP 처리까지 19개 주제가 준비되어 있습니다. 다만 대부분 Quiz와 Re-test가 남아 있어 아직 완료 상태는 아닙니다.

- [OS 첫 주제](docs/operating-systems/01-os-and-kernel.md)
- [OS 19번](docs/operating-systems/19-tcp-in-the-os.md)
- [학습 기록](progress/learning-log.md)

## 다음 네트워크 주제

Idempotency Key → Backpressure 심화 → Connection Pooling → NAT / Ephemeral Port를 정리한 뒤, Networking 복습 체크포인트를 두고 Database로 넘어갑니다.

- [전체 로드맵](ROADMAP.md)

## 언어 운영

정확한 이해를 위해 상세 학습 문서는 한국어로 먼저 작성합니다.
개념을 충분히 이해한 뒤에는 기술면접에 사용할 수 있는 짧은 영어 답변도 추가합니다.
