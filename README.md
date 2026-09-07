# Backend Interview Preparation

[한국어](README.ko.md)

A structured repository for building the computer science foundations required
for backend engineering interviews.

The repository uses a Feynman-style learning loop:

1. Explain each concept in plain language.
2. Connect it to a concrete backend example.
3. Verify understanding with recall questions and interview drills.
4. Run a small experiment when code or system observation adds value.
5. Record mistakes and improve the explanation.

## Scope

| Area | Topics |
|---|---|
| Operating Systems | Processes, threads, memory, concurrency, I/O, sockets |
| Networking | TCP/IP, UDP, DNS, HTTP, QUIC, TLS, proxies, load balancing, CDN, caching, WebSocket, SSE, gRPC, API Gateway, resilience |
| Databases | Indexes, transactions, isolation, locking, query execution |
| Runtime & Concurrency | Thread pools, async I/O, GC, synchronization |
| System Design | Caching, queues, replication, sharding, reliability |

## Learning Standard

A topic is complete only when I can:

- explain it without technical jargon;
- describe why it exists;
- explain its main trade-offs;
- connect it to backend development;
- answer follow-up questions without notes;
- reproduce the key behavior with code or system tools when appropriate.

## Progress

| Area | Current topic | Status |
|---|---|---|
| Operating Systems | 19 topics prepared; quizzes pending | Review |
| Networking | Bulkhead & Resilience Patterns | Learning |
| Databases | Not started | Pending |
| Runtime & Concurrency | Not started | Pending |
| System Design | Not started | Pending |

## Current Lesson

- [25. Bulkhead & Resilience Patterns](docs/networking/25-bulkhead-resilience-patterns.md)
- [Knowledge Check](quizzes/networking/25-bulkhead-resilience-patterns.md)

## Networking Lessons

1. [TCP/IP Layers](docs/networking/01-tcp-ip-layers.md) · [Quiz](quizzes/networking/01-tcp-ip-layers.md)
2. [TCP 3-way Handshake](docs/networking/02-tcp-three-way-handshake.md) · [Quiz](quizzes/networking/02-tcp-three-way-handshake.md)
3. [TCP Termination and TIME_WAIT](docs/networking/03-tcp-termination-time-wait.md) · [Quiz](quizzes/networking/03-tcp-termination-time-wait.md)
4. [TCP Retransmission and RTO](docs/networking/04-tcp-retransmission-rto.md) · [Quiz](quizzes/networking/04-tcp-retransmission-rto.md)
5. [TCP Flow Control and Congestion Control](docs/networking/05-flow-congestion-control.md) · [Quiz](quizzes/networking/05-flow-congestion-control.md)
6. [UDP vs TCP](docs/networking/06-udp-vs-tcp.md) · [Quiz](quizzes/networking/06-udp-vs-tcp.md)
7. [DNS](docs/networking/07-dns.md) · [Quiz](quizzes/networking/07-dns.md)
8. [HTTP/1.1](docs/networking/08-http-1-1.md) · [Quiz](quizzes/networking/08-http-1-1.md)
9. [HTTP/2](docs/networking/09-http-2.md) · [Quiz](quizzes/networking/09-http-2.md)
10. [HTTP/3 & QUIC](docs/networking/10-http-3-quic.md) · [Quiz](quizzes/networking/10-http-3-quic.md)
11. [TLS & HTTPS](docs/networking/11-tls-https.md) · [Quiz](quizzes/networking/11-tls-https.md)
12. [Certificates & PKI](docs/networking/12-certificate-pki.md) · [Quiz](quizzes/networking/12-certificate-pki.md)
13. [Forward Proxy & Reverse Proxy](docs/networking/13-forward-reverse-proxy.md) · [Quiz](quizzes/networking/13-forward-reverse-proxy.md)
14. [Load Balancing](docs/networking/14-load-balancing.md) · [Quiz](quizzes/networking/14-load-balancing.md)
15. [CDN & HTTP Cache](docs/networking/15-cdn-http-cache.md) · [Quiz](quizzes/networking/15-cdn-http-cache.md)
16. [WebSocket](docs/networking/16-websocket.md) · [Quiz](quizzes/networking/16-websocket.md)
17. [Server-Sent Events](docs/networking/17-server-sent-events.md) · [Quiz](quizzes/networking/17-server-sent-events.md)
18. [gRPC](docs/networking/18-grpc.md) · [Quiz](quizzes/networking/18-grpc.md)
19. [REST vs RPC](docs/networking/19-rest-vs-rpc.md) · [Quiz](quizzes/networking/19-rest-vs-rpc.md)
20. [API Gateway](docs/networking/20-api-gateway.md) · [Quiz](quizzes/networking/20-api-gateway.md)
21. [Rate Limiting](docs/networking/21-rate-limiting.md) · [Quiz](quizzes/networking/21-rate-limiting.md)
22. [Timeout Budget](docs/networking/22-timeout-budget.md) · [Quiz](quizzes/networking/22-timeout-budget.md)
23. [Retry, Exponential Backoff & Jitter](docs/networking/23-retry-backoff-jitter.md) · [Quiz](quizzes/networking/23-retry-backoff-jitter.md)
24. [Circuit Breaker](docs/networking/24-circuit-breaker.md) · [Quiz](quizzes/networking/24-circuit-breaker.md)
25. [Bulkhead & Resilience Patterns](docs/networking/25-bulkhead-resilience-patterns.md) · [Quiz](quizzes/networking/25-bulkhead-resilience-patterns.md)

## Operating Systems Track

The OS track contains 19 prepared topics from OS/kernel fundamentals through file descriptors, DMA, zero-copy, socket internals and TCP processing inside the OS. Most quizzes and re-tests are still pending, so the track remains in review rather than complete.

- [OS first topic](docs/operating-systems/01-os-and-kernel.md)
- [OS topic 19](docs/operating-systems/19-tcp-in-the-os.md)
- [Learning Log](progress/learning-log.md)

## Next Networking Topics

Idempotency keys, backpressure in depth, connection pooling, NAT and ephemeral ports, and a networking review checkpoint before moving into databases.

## Language Policy

Detailed learning notes are written in Korean first for accurate understanding.
Concise English interview answers will be added after each topic is understood.
