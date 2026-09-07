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

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- C 코드 또는 Linux 명령어 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Next Action

순서대로 자료 없이 답합니다.

1. [OS와 Kernel](../quizzes/operating-systems/01-os-and-kernel.md)
2. [User/Kernel Mode와 System Call](../quizzes/operating-systems/02-user-kernel-mode-system-call.md)
3. [Process와 Thread](../quizzes/operating-systems/03-process-vs-thread.md)
4. [Process Address Space](../quizzes/operating-systems/04-process-address-space.md)
5. [Context Switching](../quizzes/operating-systems/05-context-switching.md)
6. [CPU Scheduling](../quizzes/operating-systems/06-cpu-scheduling.md)
7. [Blocking / Non-blocking / Sync / Async](../quizzes/operating-systems/07-blocking-nonblocking-sync-async.md)
8. [Concurrency vs Parallelism](../quizzes/operating-systems/08-concurrency-vs-parallelism.md)
9. [Race Condition / Lock / Deadlock](../quizzes/operating-systems/09-race-lock-deadlock.md)

특히 다음 문장을 자신의 말로 설명할 수 있는지 확인합니다.

- 많은 Runnable Thread는 CPU Core 수를 늘리지 않고 Scheduling 비용만 늘릴 수 있다.
- async I/O의 핵심은 I/O 자체를 마법처럼 빠르게 만드는 것이 아니라 대기 중 Thread 점유를 줄이는 것이다.
- Concurrency와 Parallelism은 같은 개념이 아니다.
- Shared Mutable State는 Race Condition의 핵심 원인이다.
- In-process Lock은 여러 서버 Instance 사이의 DB Race Condition을 직접 해결하지 못한다.
- Deadlock의 네 조건 중 하나를 깨면 Deadlock을 예방할 수 있다.
