# Quiz — 07. Consistency / Availability

## Status

- Explanation: Prepared
- Quiz: Pending
- Re-test: Pending

## Recall Questions

1. Strong Consistency와 Eventual Consistency의 차이를 설명하라.
2. Read-after-write Consistency가 필요한 사용자 경험 예시는 무엇인가?
3. CAP Theorem을 `3개 중 2개 선택`이라고만 설명하면 왜 부족한가?
4. Network Partition 상황에서 Consistency를 우선한다는 것은 어떤 의미인가?
5. Availability를 우선하는 operation의 예를 들어라.
6. Eventual Consistency를 쓸 때 retry와 reconciliation이 필요한 이유는 무엇인가?
7. Quorum에서 N, R, W는 무엇을 의미하는가?
8. `R + W > N` 조건만으로 모든 consistency가 자동 해결되지 않는 이유는 무엇인가?
9. Distributed Consistency와 Database Isolation의 차이는 무엇인가?
10. Replica Lag과 Read-after-write의 관계를 설명하라.
11. Strong Consistency가 항상 최선이 아닌 이유는 무엇인가?
12. Eventual Consistency가 `데이터가 틀려도 된다`는 뜻이 아닌 이유는 무엇인가?

## Interview Drills

### Q1
마지막 재고 1개를 여러 사용자가 동시에 구매하려 한다. Availability와 Consistency 중 무엇을 우선하겠는가? 이유는?

### Q2
좋아요 수가 몇 초 늦게 반영돼도 괜찮은 서비스라면 어떤 consistency model을 선택할 수 있는가?

### Q3
사용자가 프로필 수정 직후 이전 값을 본다. 어떤 consistency guarantee가 부족한 것인가?

### Q4
`우리 서비스는 AP 시스템입니다`라는 말이 왜 지나치게 단순한 설명일 수 있는가?

### Q5
비동기 주문 처리에서 중간 상태가 사용자에게 보일 수 있다. 어떤 product/backend 설계가 필요한가?

## 60-second Test

자료 없이 60초 안에 다음을 포함해 설명한다.

- Strong vs Eventual
- CAP의 partition 조건
- operation별 trade-off
- Read-after-write
- stale read
- reconciliation
