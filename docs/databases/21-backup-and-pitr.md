# 21. Backup과 PITR(Point-in-Time Recovery)

## 한 줄 요약

Backup은 **데이터를 별도 시점의 복구 가능한 형태로 보관하는 것**이고, PITR은 Backup과 WAL/Transaction Log를 이용해 **특정 시점까지 데이터베이스를 되돌리는 복구 방식**입니다.

---

## 왜 Replication만으로는 부족할까?

Replica는 보통 Primary의 변경을 따라갑니다.

즉 Primary에서 실수로 다음과 같은 작업을 하면:

```sql
DELETE FROM orders;
```

그 삭제도 Replica로 복제될 수 있습니다.

따라서:

- Replication = 가용성과 읽기 확장
- Backup = 과거 상태 복구

라는 역할 차이를 구분해야 합니다.

---

## Backup의 종류

### 1. Full Backup

전체 데이터를 한 번에 저장합니다.

장점:

- 복구가 단순함

단점:

- 시간이 오래 걸림
- 저장 공간이 큼

### 2. Incremental Backup

마지막 Backup 이후 변경분만 저장합니다.

장점:

- 빠름
- 저장 공간 절약

단점:

- 복구 시 여러 Backup Chain을 따라가야 할 수 있음

### 3. Differential Backup

마지막 Full Backup 이후의 변경분을 저장합니다.

---

## PITR은 어떻게 동작할까?

예를 들어:

- 01:00 Full Backup
- 01:00~09:30 WAL 보관
- 09:17 실수로 대량 삭제

이라면:

1. 01:00 Backup을 복원하고
2. WAL을 순서대로 재생하고
3. 09:16:59 직전에서 멈추는 방식으로 복구할 수 있습니다.

핵심은:

> Backup은 기준점이고 WAL/Transaction Log는 그 이후의 변경 기록이다.

---

## RPO와 RTO

### RPO — Recovery Point Objective

얼마만큼의 데이터 손실까지 허용할 수 있는가?

예:

- RPO = 5분

이면 장애 시 최대 5분치 데이터 손실을 허용한다는 의미입니다.

### RTO — Recovery Time Objective

얼마 안에 서비스를 복구해야 하는가?

예:

- RTO = 30분

이면 30분 안에 복구 완료를 목표로 합니다.

---

## Backup이 있어도 안심하면 안 되는 이유

Backup은 **복원 가능한지 검증해야 합니다.**

백업 파일만 존재하고 실제 Restore가 실패하면 의미가 없습니다.

따라서 운영에서는:

- 정기 Restore Test
- Backup checksum / integrity 검증
- 별도 Region/Account/Object Storage 보관
- Retention 정책
- 암호화

까지 봐야 합니다.

---

## Application 장애와 연결

잘못된 Migration이나 운영자 실수로 데이터가 깨졌을 때:

- Replica Failover만으로는 해결되지 않을 수 있음
- Cache를 비운다고 해결되지 않음
- Application Rollback만으로 이미 변경된 DB를 되돌릴 수 없음

이때 Backup/PITR이 마지막 복구 수단이 됩니다.

---

## 흔한 오해

### "Replica가 있으니 Backup은 필요 없다"

틀립니다. 논리적 오류도 그대로 복제됩니다.

### "Backup 파일만 있으면 된다"

틀립니다. Restore Drill을 해보지 않은 Backup은 신뢰할 수 없습니다.

### "PITR은 항상 정확히 원하는 한 Row만 되돌린다"

아닙니다. 보통 DB 전체 또는 복구 인스턴스를 특정 시점으로 복원한 뒤 필요한 데이터를 추출하는 방식이 사용됩니다.

---

## 60초 면접 답변

> Replication은 같은 데이터를 다른 Node에 복제해 가용성과 읽기 확장성을 높이는 기술이고, Backup은 과거 상태를 별도로 보관해 논리적 삭제나 데이터 손상에서 복구하기 위한 기술입니다. PITR은 Full Backup 같은 기준점에 WAL이나 Transaction Log를 재생해서 특정 시점까지 데이터베이스를 복원합니다. 운영에서는 RPO와 RTO를 기준으로 Backup 주기와 Log 보관 전략을 정하고, 실제 Restore Test를 정기적으로 수행해야 합니다. Replica는 잘못된 DELETE 같은 논리적 오류도 따라갈 수 있기 때문에 Backup을 대체할 수 없습니다.

---

## 백엔드 연결 포인트

- Migration 실패 대응
- 운영자 실수 복구
- RPO / RTO
- Disaster Recovery
- Replica와 Backup 역할 구분
- Restore Drill

다음: **22. WAL / Checkpoint / Crash Recovery**
