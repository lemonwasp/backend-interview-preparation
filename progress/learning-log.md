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
| 2026-09-08 | OS | Memory Visibility / Memory Barrier | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Virtual Memory Deep Dive | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Paging / Page Fault | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Page Cache / File I/O | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | I/O Multiplexing | Prepared | Pending | Pending | Learning |

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- C 코드 또는 Linux 명령어 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Next Action

현재는 문서가 준비된 상태이며 완료 상태가 아닙니다. 다음 순서로 자료 없이 설명합니다.

1. Memory Visibility에서 Atomicity / Visibility / Ordering 차이
2. Virtual Address → TLB / Page Table / MMU → Physical Address 흐름
3. Page Fault가 항상 오류가 아닌 이유와 Minor / Major Fault 차이
4. Page Cache에서 read/write가 Physical Storage와 분리될 수 있는 이유
5. I/O Multiplexing이 Thread-per-connection보다 유리한 이유

특히 다음 문장을 자신의 말로 설명할 수 있어야 합니다.

- `volatile`은 복합 연산을 자동으로 Atomic하게 만들지 않는다.
- Virtual Memory는 Swap보다 훨씬 넓은 주소 추상화와 보호 메커니즘이다.
- `write()` 성공은 Storage durability 완료와 같은 의미가 아닐 수 있다.
- I/O Multiplexing은 CPU Parallelism이 아니라 많은 I/O 대기를 효율적으로 관리하는 기술이다.
