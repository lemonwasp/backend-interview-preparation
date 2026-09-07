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
| Databases | Relational model, keys, indexes, query execution, transactions, isolation, MVCC, locking |
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
| Networking | 30 topics prepared; mock interview pending | Review |
| Databases | Transaction & ACID | Learning |
| Runtime & Concurrency | Not started | Pending |
| System Design | Not started | Pending |

## Current Lesson

- [05. Transaction and ACID](docs/databases/05-transaction-and-acid.md)
- [Knowledge Check](quizzes/databases/05-transaction-and-acid.md)

## Database Lessons

1. [Relational Model and Keys](docs/databases/01-relational-model-and-keys.md) · [Quiz](quizzes/databases/01-relational-model-and-keys.md)
2. [B-Tree Index](docs/databases/02-b-tree-index.md) · [Quiz](quizzes/databases/02-b-tree-index.md)
3. [Clustered vs Non-clustered Index](docs/databases/03-clustered-vs-nonclustered-index.md) · [Quiz](quizzes/databases/03-clustered-vs-nonclustered-index.md)
4. [Query Execution and EXPLAIN](docs/databases/04-query-execution-and-explain.md) · [Quiz](quizzes/databases/04-query-execution-and-explain.md)
5. [Transaction and ACID](docs/databases/05-transaction-and-acid.md) · [Quiz](quizzes/databases/05-transaction-and-acid.md)

## Networking Track

The Networking track contains 29 concept lessons plus one 60-second-answer review and a 40-question mock interview. It remains in review until the mock interview and re-test requirements are met.

- [Networking Review](docs/networking/30-networking-review-60-second-answers.md)
- [Networking Mock Interview](quizzes/networking/30-networking-review-60-second-answers.md)

## Operating Systems Track

The OS track contains 19 prepared topics from OS/kernel fundamentals through file descriptors, DMA, zero-copy, socket internals and TCP processing inside the OS. Most quizzes and re-tests are still pending.

- [OS first topic](docs/operating-systems/01-os-and-kernel.md)
- [OS topic 19](docs/operating-systems/19-tcp-in-the-os.md)
- [Learning Log](progress/learning-log.md)

## Next Database Topics

Isolation levels, concurrency anomalies, MVCC, row/table locks, deadlocks, normalization, optimistic/pessimistic locking, replication, partitioning and database connection/transaction behavior.

## Language Policy

Detailed learning notes are written in Korean first for accurate understanding. Concise English interview answers will be added after each topic is understood.
