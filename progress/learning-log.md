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
| 2026-09-08 | Network | TCP/IP Layers | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | TCP 3-way Handshake | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | TCP Termination / TIME_WAIT | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | TCP Retransmission / RTO | Prepared | Pending | Pending | Learning |
| 2026-09-08 | Network | Flow Control / Congestion Control | Prepared | Pending | Pending | Learning |

## Evidence Rules

다음 중 하나 이상의 증거가 있어야 학습 완료로 기록합니다.

- 자신의 말로 작성한 파인만 설명
- 확인 질문 답변과 피드백
- C 코드 또는 Linux 명령어 실험 결과
- 60초 기술면접 답변
- 1일·7일 후 재시험 결과

## Current Checkpoint

OS 문서는 19개 주제까지 준비되었고 Networking 문서는 5개 주제까지 준비되었습니다. 대부분 Quiz와 Re-test가 Pending이므로 문서 생성 자체는 완료 증거가 아닙니다.

Networking에서 다음을 자료 없이 설명할 수 있어야 합니다.

1. TCP/IP에서 Application / Transport / Internet / Link 계층의 책임 차이
2. SYN → SYN/ACK → ACK가 양방향 연결과 Sequence Number를 동기화하는 과정
3. TIME_WAIT가 마지막 ACK 재전송과 오래된 Segment 격리에 필요한 이유
4. TCP가 RTO와 Duplicate ACK를 이용해 Loss를 복구하는 방식
5. Flow Control은 Receiver 보호, Congestion Control은 Network 보호라는 차이
6. `rwnd`와 `cwnd` 중 더 강한 제한이 Sender의 전송량에 영향을 준다는 점

## Next Action

1. [TCP/IP Layers Quiz](../quizzes/networking/01-tcp-ip-layers.md)
2. [TCP Handshake Quiz](../quizzes/networking/02-tcp-three-way-handshake.md)
3. [TCP Termination / TIME_WAIT Quiz](../quizzes/networking/03-tcp-termination-time-wait.md)
4. [Retransmission / RTO Quiz](../quizzes/networking/04-tcp-retransmission-rto.md)
5. [Flow / Congestion Control Quiz](../quizzes/networking/05-flow-congestion-control.md)
6. 다음 문서: UDP vs TCP → DNS → HTTP
