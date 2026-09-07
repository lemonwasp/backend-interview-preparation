# Quiz — WAL / Checkpoint / Crash Recovery

Status: **답변 대기**

## Questions

1. Write-Ahead Logging의 핵심 원칙은 무엇인가요?
2. Data Page보다 WAL을 먼저 durable하게 기록하는 이유는 무엇인가요?
3. Dirty Page란 무엇인가요?
4. Checkpoint는 왜 필요한가요?
5. Checkpoint가 Crash Recovery 시간을 줄이는 이유는 무엇인가요?
6. Commit 직후 Data Page가 아직 디스크에 쓰이지 않았는데 서버가 Crash 나면 어떻게 복구할 수 있나요?
7. Redo와 Undo를 면접 수준에서 어떻게 설명하겠습니까?
8. 모든 DBMS가 동일한 WAL/Redo/Undo 구조를 사용한다고 말하면 왜 위험한가요?
9. `write()` 성공과 durable storage 반영이 같은 의미가 아닐 수 있는 이유는 무엇인가요?
10. fsync 정책은 latency와 durability에 어떤 trade-off를 만들 수 있나요?
11. WAL이 있어도 Backup이 필요한 이유는 무엇인가요?
12. COMMIT 성공이 Replica 반영 완료를 항상 의미하지 않는 이유는 무엇인가요?
13. Checkpoint가 너무 잦을 때와 너무 드물 때 각각 어떤 문제가 있을 수 있나요?

## Interview Drill

> "Data Page가 디스크에 안 써졌는데도 COMMIT을 성공시킬 수 있다면 Durability가 깨지는 것 아닌가요?"

WAL을 이용해 60초 안에 설명해보세요.

## 평가 기준

- WAL = log-before-data 원칙을 설명하는가
- Dirty Page / Checkpoint / Recovery 흐름을 연결하는가
- COMMIT과 durable log의 관계를 설명하는가
- OS Page Cache/fsync와 연결할 수 있는가
- DBMS별 세부 구현 차이를 과도하게 일반화하지 않는가

## Evaluation

- 정확도: Pending
- 실무 연결: Pending
- 꼬리질문 대응: Pending
- 재시험 필요 여부: Pending
