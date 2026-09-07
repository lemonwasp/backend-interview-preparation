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
| Operating Systems | Processes, threads, memory, concurrency, I/O |
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

## Repository Structure

```text
.
├── README.md
├── README.ko.md
├── ROADMAP.md
├── docs/
│   └── operating-systems/
│       ├── 01-os-and-kernel.md
│       ├── 02-user-kernel-mode-system-call.md
│       ├── 03-process-vs-thread.md
│       ├── 04-process-address-space.md
│       └── 05-context-switching.md
├── quizzes/
│   └── operating-systems/
│       ├── 01-os-and-kernel.md
│       ├── 02-user-kernel-mode-system-call.md
│       ├── 03-process-vs-thread.md
│       ├── 04-process-address-space.md
│       └── 05-context-switching.md
└── progress/
    └── learning-log.md
```

## Progress

| Area | Current topic | Status |
|---|---|---|
| Operating Systems | Context Switching | Learning |
| Networking | Not started | Pending |
| Databases | Not started | Pending |
| Runtime & Concurrency | Not started | Pending |
| System Design | Not started | Pending |

## Current Lesson

- [05. Context Switching](docs/operating-systems/05-context-switching.md)
- [Knowledge Check](quizzes/operating-systems/05-context-switching.md)

### OS Lessons

1. [Operating System and Kernel](docs/operating-systems/01-os-and-kernel.md) · [Quiz](quizzes/operating-systems/01-os-and-kernel.md)
2. [User Mode, Kernel Mode and System Calls](docs/operating-systems/02-user-kernel-mode-system-call.md) · [Quiz](quizzes/operating-systems/02-user-kernel-mode-system-call.md)
3. [Process vs Thread](docs/operating-systems/03-process-vs-thread.md) · [Quiz](quizzes/operating-systems/03-process-vs-thread.md)
4. [Process Address Space](docs/operating-systems/04-process-address-space.md) · [Quiz](quizzes/operating-systems/04-process-address-space.md)
5. [Context Switching](docs/operating-systems/05-context-switching.md) · [Quiz](quizzes/operating-systems/05-context-switching.md)

## Language Policy

Detailed learning notes are written in Korean first for accurate understanding.
Concise English interview answers will be added after each topic is understood.
