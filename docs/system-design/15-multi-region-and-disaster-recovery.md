# 15. Multi-region / Disaster Recovery

## 한 줄 정의

Multi-region 설계는 서비스를 여러 지역(Region)에 분산해 **대규모 지역 장애, latency, 데이터 주권, 복구 요구사항**을 다루는 설계이고, Disaster Recovery(DR)는 큰 장애 후 서비스를 어느 시점까지 얼마나 빨리 복구할지 계획하는 체계다.

---

## 1. 왜 Multi-region을 고려하는가

대표 이유:

- Region 전체 장애 대비
- 사용자와 가까운 Region에서 latency 감소
- 데이터 주권 / 규제
- 글로벌 트래픽 분산
- DR 요구사항

하지만 비용도 크다.

- 데이터 복제
- consistency 문제
- network 비용
- 운영 복잡도
- failover 테스트
- 배포/설정 동기화

즉 "글로벌 서비스니까 무조건 Active-Active"가 아니다.

---

## 2. RPO / RTO 다시 보기

### RPO — Recovery Point Objective

얼마나 과거 시점까지의 데이터 손실을 허용할 것인가.

예:

```text
RPO = 5분
```

최악의 경우 최근 5분 데이터 손실을 허용한다는 의미다.

### RTO — Recovery Time Objective

얼마나 빨리 서비스를 복구해야 하는가.

```text
RTO = 30분
```

큰 장애 후 30분 안에 복구 목표.

DR architecture는 RPO/RTO에서 역산해야 한다.

---

## 3. Backup은 DR의 일부이지 전부가 아니다

Backup이 있어도:

- restore에 8시간
- DNS 변경에 1시간
- application config 누락
- secret/key 누락

이라면 RTO 30분을 만족할 수 없다.

따라서 DR에는 다음이 필요하다.

- 데이터 복구
- compute/environment 준비
- traffic failover
- configuration/secrets
- 운영 절차
- 검증

---

## 4. Active-Passive

```text
Region A: Active
Region B: Standby
```

평소에는 A가 트래픽을 처리하고, 장애 시 B로 전환한다.

장점:

- 데이터 쓰기 구조가 비교적 단순
- conflict가 적음
- Active-Active보다 운영 난이도 낮음

단점:

- standby 비용
- failover 시간
- standby가 실제로 동작하는지 검증 필요
- replication lag에 따른 RPO

---

## 5. Active-Active

```text
Region A: Active
Region B: Active
```

두 Region 모두 트래픽을 처리한다.

장점:

- 지역 latency 개선
- 한 Region 장애 시 다른 Region이 이미 트래픽 처리 중

비용:

- multi-writer conflict
- replication latency
- routing
- global consistency
- split brain
- 데이터 ownership

특히 write-heavy 시스템에서는 매우 어렵다.

---

## 6. Single Writer + Multi-region Read

절충안이다.

```text
Region A: Primary Write
Region B: Read Replica
Region C: Read Replica
```

읽기 latency는 줄일 수 있지만 B/C에서 write를 하려면 A로 가야 한다.

또 replication lag 때문에 stale read가 가능하다.

---

## 7. Failover는 단순 DNS 변경이 아니다

Failover 과정에는 다음이 들어간다.

1. 장애 감지
2. 실제 Region 장애인지 확인
3. 새로운 primary 결정
4. data freshness 확인
5. traffic routing 변경
6. stale instance/fencing
7. dependency 확인
8. 사용자 트래픽 점진적 증가

너무 빠른 자동 failover는 일시 network issue를 Region 장애로 오판할 위험도 있다.

---

## 8. Split Brain

두 Region이 모두 자신이 Primary라고 생각하면 서로 다른 write가 발생할 수 있다.

```text
Region A -> value = 10
Region B -> value = 20
```

network partition이 복구되면 어느 값이 맞는지 문제가 된다.

이를 막기 위해:

- consensus/quorum
- fencing
- single-writer ownership
- epoch/term

같은 메커니즘이 필요할 수 있다.

---

## 9. Data Conflict

Active-Active에서 같은 데이터를 여러 Region이 수정하면 conflict가 생긴다.

해결 방식은 데이터 특성에 따라 다르다.

예:

- last-write-wins
- version conflict
- merge
- CRDT 계열
- region ownership
- central coordinator

금융 거래처럼 conflict resolution이 민감한 데이터는 단순 LWW가 적합하지 않을 수 있다.

---

## 10. Global Traffic Routing

사용자를 어느 Region으로 보낼지 정해야 한다.

기준:

- latency
- health
- geography
- data residency
- capacity

방법 예:

- DNS-based routing
- Global Load Balancer / Anycast 계열

하지만 routing과 data location이 따로 놀면 cross-region call이 늘어 latency와 비용이 증가한다.

---

## 11. Data Locality

사용자는 Tokyo Region에 있지만 데이터 Source of Truth가 Europe에 있으면 매 요청이 장거리 network를 탈 수 있다.

그래서 다음을 함께 본다.

- user affinity
- tenant home region
- data residency
- replication topology

Region 선택은 compute만의 문제가 아니다.

---

## 12. Dependency도 Multi-region이어야 하는가

App만 두 Region에 두고 DB가 한 Region이면 DB 장애에 취약하다.

반대로 모든 dependency를 multi-region으로 만들면 비용이 매우 커진다.

따라서 dependency별로:

- criticality
- recovery strategy
- RPO/RTO

를 정한다.

예:

```text
Payment DB: warm standby
Recommendation Cache: rebuild 가능
Analytics: RTO 24h 허용
```

---

## 13. DR Runbook

실제 장애 때 사람이 무엇을 해야 하는지 문서화한다.

예:

1. Region health 확인
2. write freeze 여부 결정
3. replica freshness 확인
4. standby promotion
5. traffic switch
6. smoke test
7. traffic ramp-up
8. backlog/reconciliation 확인

Runbook이 없으면 긴급 상황에서 즉흥 판단이 늘어난다.

---

## 14. DR Drill

DR은 문서만 있으면 의미가 없다.

정기적으로:

- backup restore
- standby promotion
- region failover
- DNS/global routing change
- secret/config validation

을 실제로 연습해야 한다.

이를 통해 실제 RTO를 측정한다.

---

## 15. Failback

장애 Region이 복구된 뒤 원래 Region으로 돌아가는 과정도 어렵다.

확인할 것:

- 장애 중 발생한 신규 데이터
- replication direction
- conflict
- traffic 이동
- old primary fencing

Failover보다 Failback이 더 위험할 수 있다.

---

## 16. 비용과 단계적 접근

모든 서비스가 Active-Active Multi-region을 필요로 하지는 않는다.

단계 예:

```text
1. Backup + Restore Test
2. Multi-AZ
3. Cross-region Backup
4. Warm Standby
5. Active-Passive
6. 필요 시 Active-Active
```

비즈니스 SLO/RPO/RTO에 맞춰 올라간다.

---

## 17. 면접에서 반드시 언급할 것

Multi-region 설계를 제안했다면 다음 질문에 답해야 한다.

- 왜 필요한가?
- RPO/RTO는?
- write topology는?
- conflict는 어떻게 처리하는가?
- failover detection은?
- split brain은?
- failback은?
- 실제 DR drill을 하는가?

---

## 18. 흔한 오해

### 오해 1: Multi-region이면 무조건 HA다

아니다. 공통 dependency나 잘못된 failover가 있으면 같이 실패할 수 있다.

### 오해 2: Active-Active가 가장 좋은 구조다

가장 복잡하고 비용도 크다. 필요성이 있어야 한다.

### 오해 3: Backup만 있으면 DR이 된다

Restore 시간, traffic routing, configuration까지 포함해야 한다.

### 오해 4: Failover만 설계하면 된다

Failback과 reconciliation도 필요하다.

---

## 19. 60초 기술면접 답변

> Multi-region은 Region 단위 장애나 글로벌 latency 요구를 해결할 수 있지만 데이터 복제와 consistency 비용이 크기 때문에 RPO와 RTO부터 정하고 설계해야 합니다. 단순한 구조는 Active-Passive로 한 Region이 write를 담당하고 다른 Region을 standby로 두는 방식이고, Active-Active는 latency와 failover 측면의 장점이 있지만 multi-writer conflict와 split brain을 해결해야 합니다. Failover는 장애 감지, replica freshness, primary promotion, fencing, traffic switching까지 포함하며 복구 후 failback과 reconciliation도 필요합니다. 또한 DR은 backup 보유가 아니라 restore와 region failover를 실제로 정기 테스트해서 목표 RTO를 검증해야 합니다.
