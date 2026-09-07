# Backend Interview Preparation

[한국어](README.ko.md)

A structured repository for building the computer science foundations required for backend engineering interviews.

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
| Networking | TCP/IP, UDP, DNS, HTTP, QUIC, TLS, proxies, load balancing, realtime transport, RPC, gateways, resilience, connection management |
| Databases | Relational model, indexes, query execution, transactions, isolation, MVCC, locking, normalization, constraints, replication, sharding, connection pools, ORM, migrations, pagination, caching, backup, recovery, CDC |
| Runtime & Concurrency | CLR/runtime model, thread pools, Task, async/await, synchronization, concurrent collections, cancellation, GC, allocation, memory/resource lifetime, async streams, context flow, observability |
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
| Networking | 30 topics prepared; mock interview pending | Review |
| Databases | 25 topics prepared; mock interview pending | Review |
| Runtime & Concurrency | Runtime Observability | Learning |
| System Design | Not started | Pending |

## Current Lesson

- [15. Runtime Observability](docs/runtime-concurrency/15-runtime-observability.md)
- [Knowledge Check](quizzes/runtime-concurrency/15-runtime-observability.md)

## Runtime & Concurrency Lessons

1. [Runtime Process Model](docs/runtime-concurrency/01-runtime-process-model.md) · [Quiz](quizzes/runtime-concurrency/01-runtime-process-model.md)
2. [Thread Pool](docs/runtime-concurrency/02-thread-pool.md) · [Quiz](quizzes/runtime-concurrency/02-thread-pool.md)
3. [Task / Future / Promise](docs/runtime-concurrency/03-task-future-promise.md) · [Quiz](quizzes/runtime-concurrency/03-task-future-promise.md)
4. [async / await Internals](docs/runtime-concurrency/04-async-await-internals.md) · [Quiz](quizzes/runtime-concurrency/04-async-await-internals.md)
5. [Synchronization Primitives](docs/runtime-concurrency/05-synchronization-primitives.md) · [Quiz](quizzes/runtime-concurrency/05-synchronization-primitives.md)
6. [Concurrent Collections](docs/runtime-concurrency/06-concurrent-collections.md) · [Quiz](quizzes/runtime-concurrency/06-concurrent-collections.md)
7. [Cancellation and Timeout](docs/runtime-concurrency/07-cancellation-and-timeout.md) · [Quiz](quizzes/runtime-concurrency/07-cancellation-and-timeout.md)
8. [GC Generations](docs/runtime-concurrency/08-gc-generations.md) · [Quiz](quizzes/runtime-concurrency/08-gc-generations.md)
9. [Allocation and Boxing](docs/runtime-concurrency/09-allocation-and-boxing.md) · [Quiz](quizzes/runtime-concurrency/09-allocation-and-boxing.md)
10. [Managed Memory Leak](docs/runtime-concurrency/10-managed-memory-leak.md) · [Quiz](quizzes/runtime-concurrency/10-managed-memory-leak.md)
11. [IDisposable and Resource Lifetime](docs/runtime-concurrency/11-idisposable-resource-lifetime.md) · [Quiz](quizzes/runtime-concurrency/11-idisposable-resource-lifetime.md)
12. [Async Streams and Channels](docs/runtime-concurrency/12-async-streams-and-channels.md) · [Quiz](quizzes/runtime-concurrency/12-async-streams-and-channels.md)
13. [ThreadPool Starvation Diagnostics](docs/runtime-concurrency/13-threadpool-starvation-diagnostics.md) · [Quiz](quizzes/runtime-concurrency/13-threadpool-starvation-diagnostics.md)
14. [ExecutionContext and Context Flow](docs/runtime-concurrency/14-executioncontext-context-flow.md) · [Quiz](quizzes/runtime-concurrency/14-executioncontext-context-flow.md)
15. [Runtime Observability](docs/runtime-concurrency/15-runtime-observability.md) · [Quiz](quizzes/runtime-concurrency/15-runtime-observability.md)

## Database Track

The Database track contains 24 concept lessons plus one 60-second-answer review and a 40-question mock interview. It remains in review until the mock interview and re-test requirements are met.

- [Database Review](docs/databases/25-database-review-60-second-answers.md)
- [Database Mock Interview](quizzes/databases/25-database-review-60-second-answers.md)

## Networking Track

The Networking track contains 29 concept lessons plus one 60-second-answer review and a 40-question mock interview. It remains in review until the mock interview and re-test requirements are met.

- [Networking Review](docs/networking/30-networking-review-60-second-answers.md)
- [Networking Mock Interview](quizzes/networking/30-networking-review-60-second-answers.md)

## Operating Systems Track

The OS track contains 19 prepared topics from OS/kernel fundamentals through file descriptors, DMA, zero-copy, socket internals and TCP processing inside the OS. Most quizzes and re-tests are still pending.

- [OS first topic](docs/operating-systems/01-os-and-kernel.md)
- [OS topic 19](docs/operating-systems/19-tcp-in-the-os.md)
- [Learning Log](progress/learning-log.md)

## Next Runtime Topic

Create a Runtime & Concurrency review checkpoint and mock interview, then move new-topic preparation to System Design. Existing OS, Networking, Database and Runtime topics remain Prepared until quizzes and re-tests are passed.

## Language Policy

Detailed learning notes are written in Korean first for accurate understanding. Concise English interview answers will be added after each topic is understood.
