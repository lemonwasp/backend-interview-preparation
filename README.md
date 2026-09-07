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
| Networking | TCP/IP, HTTP, DNS, sockets, load balancing |
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
| Networking | Flow Control & Congestion Control | Learning |
| Databases | Not started | Pending |
| Runtime & Concurrency | Not started | Pending |
| System Design | Not started | Pending |

## Current Lesson

- [05. TCP Flow Control and Congestion Control](docs/networking/05-flow-congestion-control.md)
- [Knowledge Check](quizzes/networking/05-flow-congestion-control.md)

## Networking Lessons

1. [TCP/IP Layers](docs/networking/01-tcp-ip-layers.md) · [Quiz](quizzes/networking/01-tcp-ip-layers.md)
2. [TCP 3-way Handshake](docs/networking/02-tcp-three-way-handshake.md) · [Quiz](quizzes/networking/02-tcp-three-way-handshake.md)
3. [TCP Termination and TIME_WAIT](docs/networking/03-tcp-termination-time-wait.md) · [Quiz](quizzes/networking/03-tcp-termination-time-wait.md)
4. [TCP Retransmission and RTO](docs/networking/04-tcp-retransmission-rto.md) · [Quiz](quizzes/networking/04-tcp-retransmission-rto.md)
5. [TCP Flow Control and Congestion Control](docs/networking/05-flow-congestion-control.md) · [Quiz](quizzes/networking/05-flow-congestion-control.md)

## Operating Systems Track

The OS track currently contains 19 prepared topics from OS/kernel fundamentals through file descriptors, DMA, zero-copy, socket internals and TCP processing inside the OS. The documents are prepared, but most quizzes and re-tests are still pending, so the track is not marked complete.

Start review from:

- [01. Operating System and Kernel](docs/operating-systems/01-os-and-kernel.md)
- [19. TCP Processing Inside the OS](docs/operating-systems/19-tcp-in-the-os.md)
- [Learning Log](progress/learning-log.md)

## Next Networking Topics

UDP vs TCP, DNS, HTTP/1.1, HTTP/2, HTTP/3 and QUIC, TLS, proxies, reverse proxies and load balancing.

## Language Policy

Detailed learning notes are written in Korean first for accurate understanding.
Concise English interview answers will be added after each topic is understood.
