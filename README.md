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
| Runtime & Concurrency | CLR/runtime model, thread pools, Task, async/await, synchronization, GC, allocation, concurrent collections |
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
| Runtime & Concurrency | Synchronization Primitives | Learning |
| System Design | Not started | Pending |

## Current Lesson

- [05. Synchronization Primitives](docs/runtime-concurrency/05-synchronization-primitives.md)
- [Knowledge Check](quizzes/runtime-concurrency/05-synchronization-primitives.md)

## Runtime & Concurrency Lessons

1. [Runtime Process Model](docs/runtime-concurrency/01-runtime-process-model.md) · [Quiz](quizzes/runtime-concurrency/01-runtime-process-model.md)
2. [Thread Pool](docs/runtime-concurrency/02-thread-pool.md) · [Quiz](quizzes/runtime-concurrency/02-thread-pool.md)
3. [Task / Future / Promise](docs/runtime-concurrency/03-task-future-promise.md) · [Quiz](quizzes/runtime-concurrency/03-task-future-promise.md)
4. [async / await Internals](docs/runtime-concurrency/04-async-await-internals.md) · [Quiz](quizzes/runtime-concurrency/04-async-await-internals.md)
5. [Synchronization Primitives](docs/runtime-concurrency/05-synchronization-primitives.md) · [Quiz](quizzes/runtime-concurrency/05-synchronization-primitives.md)

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

## Next Runtime Topics

Concurrent collections, cancellation and timeouts, GC generations, allocation/boxing, managed memory leaks, `IDisposable` and resource lifetime, async streams/channels, ThreadPool starvation diagnostics and runtime observability.

## Language Policy

Detailed learning notes are written in Korean first for accurate understanding. Concise English interview answers will be added after each topic is understood.
