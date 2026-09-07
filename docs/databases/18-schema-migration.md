# 18. Schema Migration

## 한 줄 핵심

Schema Migration은 Table 구조를 바꾸는 작업이지만, 운영 환경에서는 SQL 한 줄보다 `구버전 App과 신버전 App이 동시에 존재하는 배포 과정`까지 포함해 설계해야 한다.

## 위험한 예

현재 App이 `users.name`을 읽고 있는데 Migration에서 즉시 Column을 삭제하면 구버전 Instance가 아직 떠 있는 동안 장애가 발생할 수 있다.

그래서 운영 Migration은 Application Deployment와 함께 생각한다.

## Expand → Migrate → Contract

### 1. Expand

새 구조를 추가하되 기존 구조는 유지한다.

```text
old_column 유지
new_column 추가
```

구/신버전 App이 모두 동작할 수 있게 한다.

### 2. Migrate

기존 데이터를 새 구조로 Backfill하고, 필요하면 일정 기간 Dual Write/Read 전환을 수행한다.

### 3. Contract

모든 App이 새 구조만 사용한다는 것이 확인된 뒤 old column/index/table을 제거한다.

## 큰 Table의 ALTER TABLE

큰 Table에서 Column/Index 변경은 DBMS와 변경 종류에 따라:

- Table Rewrite
- 장시간 Lock
- I/O 폭증
- Replication Lag
- Transaction Log/WAL 증가

를 만들 수 있다.

따라서 `DDL은 항상 즉시 끝난다`고 가정하면 안 된다.

## Index 생성

운영 DB에서 큰 Index 생성은 CPU/I/O를 크게 사용할 수 있고 Write latency에 영향을 줄 수 있다. 일부 DBMS는 online/concurrent index build 기능을 제공하지만 구체적인 제약은 제품마다 다르다.

## Backfill

수천만 Row를 한 Transaction에서 UPDATE하면:

- Lock/MVCC version 증가
- WAL/Redo 증가
- Replica Lag
- Long Transaction

문제가 생길 수 있다.

그래서 보통 작은 Batch로 나누고 progress를 기록하며 throttle한다.

## NOT NULL 변경

새 Column을 곧바로 `NOT NULL`로 추가하면서 기존 모든 Row를 동시에 채우려고 하면 위험할 수 있다.

안전한 흐름 예:

```text
nullable column 추가
→ 새 App부터 값 기록
→ 기존 Row backfill
→ null 없음 검증
→ NOT NULL constraint 적용
```

## Dual Write의 위험

Application에서 old/new storage를 둘 다 업데이트하면 둘 중 하나만 성공하는 partial failure가 생길 수 있다.

따라서 Dual Write가 필요하다면:

- 같은 DB Transaction 안에서 가능한지
- retry/idempotency가 있는지
- reconciliation 방법이 있는지

를 고려해야 한다.

## Rollback이 항상 가능한가?

아니다.

Data를 삭제하거나 변환한 뒤 원래 정보를 잃었다면 단순 rollback으로 복원되지 않는다.

그래서 Migration의 목표는 `Rollback SQL 준비`만이 아니라 **Backward-compatible rollout**을 가능하게 만드는 것이다.

## Migration Checklist

- 구버전 App과 호환되는가?
- Lock/Rebuild 가능성이 있는가?
- Table 크기는?
- 예상 WAL/Redo/Replica Lag은?
- Backfill은 batch인가?
- 실패 중간 상태에서 재시작 가능한가?
- Monitoring과 Abort 기준이 있는가?

## 60초 면접 답변

운영 환경의 Schema Migration은 단순 DDL 실행이 아니라 Application rollout과 함께 봐야 합니다. 저는 기본적으로 Expand-Migrate-Contract 방식을 생각합니다. 먼저 새 Column이나 Table을 기존 구조와 호환되게 추가하고, 새 App이 사용하도록 전환하면서 기존 데이터를 작은 Batch로 Backfill한 다음, 모든 Instance가 새 구조만 사용한다는 것이 확인되면 옛 구조를 제거합니다. 큰 Table의 ALTER나 Index 생성은 Lock, I/O, WAL 증가와 Replica Lag을 만들 수 있기 때문에 Table 크기와 DBMS별 online DDL 특성을 확인하고 모니터링하면서 진행해야 합니다.
