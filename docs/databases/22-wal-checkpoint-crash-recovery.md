# 22. WAL / Checkpoint / Crash Recovery

## 한 줄 요약

Database는 변경 내용을 먼저 WAL(Write-Ahead Log)에 기록하고, Checkpoint로 디스크 상태를 정리해두며, Crash 후에는 WAL을 재생해서 **Commit된 변경은 살리고 미완료 변경은 정리**합니다.

---

## 왜 Data Page를 매번 즉시 디스크에 쓰지 않을까?

Memory의 Buffer Pool에 있는 Page가 수정될 때마다 실제 Data File까지 즉시 동기식으로 쓰면 너무 느립니다.

그래서 Database는 보통:

1. Memory에서 Page를 수정하고
2. 변경 내용을 순차적인 Log에 먼저 기록하고
3. 이후 Data Page를 천천히 디스크로 내립니다.

이 원칙이 WAL입니다.

> Data Page보다 복구에 필요한 Log가 먼저 durable 해야 한다.

---

## WAL의 장점

### 1. Sequential Write

Log는 대체로 순차적으로 append되므로 Random Data Page Write보다 유리합니다.

### 2. Crash Recovery

Commit은 끝났지만 Data Page가 아직 디스크에 반영되지 않았더라도 Log를 이용해 복구할 수 있습니다.

### 3. Replication / PITR 기반

DBMS에 따라 WAL/Redo/Binlog 등의 Log는 Replication이나 PITR에도 활용됩니다.

단, 각 DBMS의 Log 종류와 역할은 완전히 같지 않으므로 이름만 보고 동일하다고 단정하면 안 됩니다.

---

## Checkpoint란?

Checkpoint는 쉽게 말해:

> "여기까지는 Data Page 상태를 꽤 정리해놨다"

라는 복구 기준점을 만드는 작업입니다.

Checkpoint가 없다면 Crash Recovery 때 아주 오래된 Log부터 전부 확인해야 할 수 있습니다.

Checkpoint는:

- Dirty Page Flush
- Recovery 시작점 단축
- WAL 재생 범위 감소

에 도움을 줍니다.

---

## Dirty Page

Memory Buffer에서 수정됐지만 아직 Data File에 반영되지 않은 Page입니다.

예:

```text
Disk Data Page: balance = 100
Memory Dirty Page: balance = 80
WAL: balance 100 -> 80 기록됨
```

이 상태에서 Crash가 나더라도 WAL이 안전하게 기록돼 있다면 복구할 수 있습니다.

---

## Crash Recovery의 직관

Crash 후 Database는 Log와 Data Page 상태를 비교해 필요한 작업을 수행합니다.

DBMS마다 알고리즘은 다르지만 면접 수준에서 중요한 개념은:

- Commit된 Transaction의 변경이 Data File에 없다면 다시 적용할 수 있음(Redo)
- 미완료 Transaction의 영향은 제거되어야 함(Undo 또는 MVCC/Log 기반 정리)

입니다.

### 중요한 점

모든 DBMS가 정확히 동일한 Redo/Undo 절차를 쓰는 것은 아닙니다.

---

## COMMIT은 언제 성공했다고 말할 수 있을까?

일반적인 durability 관점에서는 필요한 Log가 안정적인 저장장치에 기록됐다는 보장이 중요합니다.

하지만 설정에 따라:

- fsync 정책
- synchronous_commit
- storage cache
- replication acknowledgement

등이 Durability와 Latency trade-off를 바꿀 수 있습니다.

즉:

> COMMIT latency가 낮아졌다면 어떤 durability guarantee를 포기했는지도 봐야 한다.

---

## Checkpoint가 너무 잦거나 너무 드물면?

### 너무 잦으면

- Disk I/O 증가
- Write burst 가능

### 너무 드물면

- Dirty Page 증가
- Recovery 시간이 길어질 수 있음
- WAL 보관량 증가 가능

따라서 workload에 맞는 균형이 필요합니다.

---

## WAL과 OS Page Cache

DB는 자체 Buffer Pool을 사용하면서 OS Page Cache와 상호작용할 수 있습니다.

또한 "write()가 성공했다"와 "전원이 꺼져도 데이터가 남는다"는 같은 의미가 아닐 수 있습니다.

그래서 fsync 계열의 durability boundary가 중요합니다.

이는 앞서 OS의 Page Cache/File I/O 주제와 연결됩니다.

---

## 장애 시나리오

### 상황

결제 Transaction이 COMMIT 성공 응답을 반환한 직후 DB 서버 전원이 꺼졌습니다.

질문:

- Data Page가 디스크에 아직 없었다면 결제 기록은 사라질까?

답:

정상적인 durability 설정이라면 Commit에 필요한 WAL이 durable하게 기록됐으므로 Crash Recovery에서 Redo되어야 합니다.

---

## 흔한 오해

### "WAL이 있으면 Backup은 필요 없다"

아닙니다. WAL만 무한정 보관하는 것이 일반적인 Backup 전략은 아닙니다. Base Backup과 결합해 PITR을 구성합니다.

### "Checkpoint는 모든 변경을 완전히 저장하고 Log를 없애는 것"

과도한 단순화입니다. 정확한 처리 방식은 DBMS마다 다릅니다.

### "COMMIT은 항상 Replica까지 반영됐다는 뜻"

아닙니다. Replication mode와 acknowledgement 정책에 따라 다릅니다.

---

## 60초 면접 답변

> WAL은 Data Page를 디스크에 쓰기 전에 복구에 필요한 변경 로그를 먼저 durable하게 기록하는 원칙입니다. 그래서 Commit 직후 서버가 Crash 나서 Data Page가 아직 반영되지 않았더라도 WAL을 재생해 Commit된 변경을 복구할 수 있습니다. Checkpoint는 Dirty Page를 정리하고 복구 기준점을 만들어 Crash Recovery 때 확인해야 할 Log 범위를 줄입니다. 다만 WAL, Redo, Undo, Binlog 같은 구조와 Commit durability 정책은 DBMS마다 세부 구현이 다르기 때문에 공통 원리와 제품별 구현을 구분해서 봐야 합니다.

---

## 백엔드 연결 포인트

- COMMIT durability
- Crash Recovery
- fsync / storage latency
- Checkpoint I/O spike
- PITR
- Replication log
- OS Page Cache와의 연결

다음: **23. Database Failure Scenarios**
