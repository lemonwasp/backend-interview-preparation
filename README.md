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
| System Design | Requirements, capacity estimation, stateless services, caching, queues, load balancing, database scaling, consistency, idempotency, rate limiting, distributed locks, service boundaries, SLOs, graceful degradation, failure modes, multi-region and disaster recovery |

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
| Runtime & Concurrency | 16 topics prepared; mock interview pending | Review |
| System Design | 16 topics prepared; mock interview pending | Review |

## Current Lesson

- [16. System Design Review — 60-second Answers & Trade-offs](docs/system-design/16-system-design-review-60-second-answers.md)
- [System Design Mock Interview](quizzes/system-design/16-system-design-review-60-second-answers.md)

## System Design Lessons

1. [Requirements and Capacity Estimation](docs/system-design/01-requirements-and-capacity-estimation.md) · [Quiz](quizzes/system-design/01-requirements-and-capacity-estimation.md)
2. [Stateless Service](docs/system-design/02-stateless-service.md) · [Quiz](quizzes/system-design/02-stateless-service.md)
3. [Cache Design](docs/system-design/03-cache-design.md) · [Quiz](quizzes/system-design/03-cache-design.md)
4. [Message Queue](docs/system-design/04-message-queue.md) · [Quiz](quizzes/system-design/04-message-queue.md)
5. [Load Balancing and Horizontal Scaling](docs/system-design/05-load-balancing-horizontal-scaling.md) · [Quiz](quizzes/system-design/05-load-balancing-horizontal-scaling.md)
6. [Database Scaling and Read/Write Patterns](docs/system-design/06-database-scaling-read-write-patterns.md) · [Quiz](quizzes/system-design/06-database-scaling-read-write-patterns.md)
7. [Consistency and Availability](docs/system-design/07-consistency-and-availability.md) · [Quiz](quizzes/system-design/07-consistency-and-availability.md)
8. [Distributed Idempotency](docs/system-design/08-distributed-idempotency.md) · [Quiz](quizzes/system-design/08-distributed-idempotency.md)
9. [Rate Limiting](docs/system-design/09-rate-limiting.md) · [Quiz](quizzes/system-design/09-rate-limiting.md)
10. [Distributed Lock](docs/system-design/10-distributed-lock.md) · [Quiz](quizzes/system-design/10-distributed-lock.md)
11. [Service Boundary and Ownership](docs/system-design/11-service-boundary-and-ownership.md) · [Quiz](quizzes/system-design/11-service-boundary-and-ownership.md)
12. [Observability and SLO](docs/system-design/12-observability-and-slo.md) · [Quiz](quizzes/system-design/12-observability-and-slo.md)
13. [Graceful Degradation](docs/system-design/13-graceful-degradation.md) · [Quiz](quizzes/system-design/13-graceful-degradation.md)
14. [Failure Mode Reasoning](docs/system-design/14-failure-mode-reasoning.md) · [Quiz](quizzes/system-design/14-failure-mode-reasoning.md)
15. [Multi-region and Disaster Recovery](docs/system-design/15-multi-region-and-disaster-recovery.md) · [Quiz](quizzes/system-design/15-multi-region-and-disaster-recovery.md)
16. [System Design Review — 60-second Answers & Trade-offs](docs/system-design/16-system-design-review-60-second-answers.md) · [Mock Interview](quizzes/system-design/16-system-design-review-60-second-answers.md)

## Runtime & Concurrency Track

The Runtime & Concurrency track contains 15 concept lessons plus one 60-second-answer review and a 40-question mock interview. It remains in review until the mock interview and re-test requirements are met.

- [Runtime & Concurrency Review](docs/runtime-concurrency/16-runtime-concurrency-review-60-second-answers.md)
- [Runtime & Concurrency Mock Interview](quizzes/runtime-concurrency/16-runtime-concurrency-review-60-second-answers.md)

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

## Review Phase

New concept expansion is paused. The repository now moves into evidence-based review: quizzes, mock interviews, explanation without notes, and D+1/D+7 re-tests. Prepared material is not treated as Completed until those checks are passed.

## Language Policy

Detailed learning notes are written in Korean first for accurate understanding. Concise English interview answers will be added after each topic is understood.
