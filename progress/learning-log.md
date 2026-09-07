# Learning Log

학습 완료 여부는 문서를 읽은 시점이 아니라, 확인 질문과 재시험 결과로 판단합니다.

| Date | Area | Topic | Explanation | Quiz | Re-test | Status |
|---|---|---|---|---|---|---|
| 2026-09-05 | OS | OS and Kernel | Completed | Pending | Pending | Learning |
| 2026-09-07 | OS | User/Kernel Mode & System Calls | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | Process vs Thread | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | Process Address Space | Prepared | Pending | Pending | Learning |
| 2026-09-07 | OS | Context Switching | Prepared | Pending | Pending | Learning |

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

특히 다음 세 문장을 자신의 말로 설명할 수 있는지 확인합니다.

- 같은 Process의 Thread는 Heap을 공유하지만 Stack은 각자 가진다.
- 같은 Virtual Address라도 Process마다 다른 Physical Page에 매핑될 수 있다.
- System Call의 Mode Switch와 Thread 간 Context Switch는 같은 개념이 아니다.
