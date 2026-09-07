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
| 2026-09-08 | OS | File Descriptor | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Interrupt / DMA | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Zero-copy / sendfile | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | Socket Internals | Prepared | Pending | Pending | Learning |
| 2026-09-08 | OS | TCP Processing Inside the OS | Prepared | Pending | Pending | Learning |

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- C 코드 또는 Linux 명령어 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Current Checkpoint

OS 문서는 19개 주제까지 준비되었습니다. 아직 대부분 Quiz와 Re-test가 Pending이므로 문서 생성 자체를 학습 완료로 간주하지 않습니다.

다음 다섯 문장을 자료 없이 설명할 수 있어야 OS → Networking 전환이 가능합니다.

1. File Descriptor는 Process-local integer handle이며 Kernel resource 자체가 아니다.
2. DMA는 NIC/Storage와 RAM 사이의 데이터 전송에서 CPU의 직접 복사 부담을 줄인다.
3. Zero-copy는 물리적 데이터 이동이 0이라는 뜻이 아니라 불필요한 CPU-mediated copy를 줄이는 최적화다.
4. `send()` 성공은 상대 애플리케이션의 수신 완료와 같은 의미가 아니다.
5. TCP 수신 경로는 크게 NIC → DMA → Kernel TCP/IP stack → Socket buffer → Application으로 설명할 수 있다.

## Next Action

1. [File Descriptor Quiz](../quizzes/operating-systems/15-file-descriptor.md)
2. [Interrupt / DMA Quiz](../quizzes/operating-systems/16-interrupt-and-dma.md)
3. [Zero-copy / sendfile Quiz](../quizzes/operating-systems/17-zero-copy-and-sendfile.md)
4. [Socket Internals Quiz](../quizzes/operating-systems/18-socket-internals.md)
5. [TCP Processing Quiz](../quizzes/operating-systems/19-tcp-in-the-os.md)
6. 이후 Networking Fundamentals로 진행
