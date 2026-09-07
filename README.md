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
| Operating Systems | TCP processing inside the OS | Learning |
| Networking | Next | Pending |
| Databases | Not started | Pending |
| Runtime & Concurrency | Not started | Pending |
| System Design | Not started | Pending |

## Current Lesson

- [19. TCP Processing Inside the OS](docs/operating-systems/19-tcp-in-the-os.md)
- [Knowledge Check](quizzes/operating-systems/19-tcp-in-the-os.md)

## OS Lessons

1. [Operating System and Kernel](docs/operating-systems/01-os-and-kernel.md) · [Quiz](quizzes/operating-systems/01-os-and-kernel.md)
2. [User Mode, Kernel Mode and System Calls](docs/operating-systems/02-user-kernel-mode-system-call.md) · [Quiz](quizzes/operating-systems/02-user-kernel-mode-system-call.md)
3. [Process vs Thread](docs/operating-systems/03-process-vs-thread.md) · [Quiz](quizzes/operating-systems/03-process-vs-thread.md)
4. [Process Address Space](docs/operating-systems/04-process-address-space.md) · [Quiz](quizzes/operating-systems/04-process-address-space.md)
5. [Context Switching](docs/operating-systems/05-context-switching.md) · [Quiz](quizzes/operating-systems/05-context-switching.md)
6. [CPU Scheduling](docs/operating-systems/06-cpu-scheduling.md) · [Quiz](quizzes/operating-systems/06-cpu-scheduling.md)
7. [Blocking, Non-blocking, Sync and Async](docs/operating-systems/07-blocking-nonblocking-sync-async.md) · [Quiz](quizzes/operating-systems/07-blocking-nonblocking-sync-async.md)
8. [Concurrency vs Parallelism](docs/operating-systems/08-concurrency-vs-parallelism.md) · [Quiz](quizzes/operating-systems/08-concurrency-vs-parallelism.md)
9. [Race Condition, Lock and Deadlock](docs/operating-systems/09-race-condition-lock-deadlock.md) · [Quiz](quizzes/operating-systems/09-race-condition-lock-deadlock.md)
10. [Memory Visibility and Memory Barriers](docs/operating-systems/10-memory-visibility-memory-barrier.md) · [Quiz](quizzes/operating-systems/10-memory-visibility-memory-barrier.md)
11. [Virtual Memory Deep Dive](docs/operating-systems/11-virtual-memory-deep-dive.md) · [Quiz](quizzes/operating-systems/11-virtual-memory-deep-dive.md)
12. [Paging and Page Faults](docs/operating-systems/12-paging-page-fault.md) · [Quiz](quizzes/operating-systems/12-paging-page-fault.md)
13. [Page Cache and File I/O](docs/operating-systems/13-page-cache-file-io.md) · [Quiz](quizzes/operating-systems/13-page-cache-file-io.md)
14. [I/O Multiplexing](docs/operating-systems/14-io-multiplexing.md) · [Quiz](quizzes/operating-systems/14-io-multiplexing.md)
15. [File Descriptor](docs/operating-systems/15-file-descriptor.md) · [Quiz](quizzes/operating-systems/15-file-descriptor.md)
16. [Interrupt and DMA](docs/operating-systems/16-interrupt-and-dma.md) · [Quiz](quizzes/operating-systems/16-interrupt-and-dma.md)
17. [Zero-copy and sendfile](docs/operating-systems/17-zero-copy-and-sendfile.md) · [Quiz](quizzes/operating-systems/17-zero-copy-and-sendfile.md)
18. [Socket Internals](docs/operating-systems/18-socket-internals.md) · [Quiz](quizzes/operating-systems/18-socket-internals.md)
19. [TCP Processing Inside the OS](docs/operating-systems/19-tcp-in-the-os.md) · [Quiz](quizzes/operating-systems/19-tcp-in-the-os.md)

## Next Track

Networking Fundamentals: TCP/IP layers, TCP handshake and teardown, retransmission and congestion control, DNS, HTTP/1.1–HTTP/3, TLS, proxies and load balancing.

## Language Policy

Detailed learning notes are written in Korean first for accurate understanding.
Concise English interview answers will be added after each topic is understood.
