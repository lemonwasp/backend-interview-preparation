# Quiz — 06. Database Scaling / Read-Write Patterns

## Status

- Explanation: Prepared
- Quiz: Pending
- Re-test: Pending

## Recall Questions

1. Database Scaling 전에 Read RPS와 Write RPS를 분리해서 봐야 하는 이유는 무엇인가?
2. Vertical Scaling의 장점과 한계는 무엇인가?
3. Read Replica는 어떤 종류의 병목을 줄이는 데 유리한가?
4. Replica Lag이 만드는 대표적인 사용자 경험 문제는 무엇인가?
5. read-after-write consistency가 필요한 예를 하나 들어라.
6. Cache가 DB 부하를 줄이는 대신 새로 만드는 문제는 무엇인가?
7. Write Scaling이 Read Scaling보다 일반적으로 어려운 이유는 무엇인가?
8. Hot Row란 무엇인가?
9. Partitioning과 Sharding의 차이를 설명하라.
10. Sharding의 대표적인 운영 비용을 4개 이상 말하라.
11. `instance count × pool size`를 계산해야 하는 이유는 무엇인가?
12. 주문 시스템에서 상품 목록 조회와 주문 생성의 DB routing을 다르게 설계할 수 있는 이유는 무엇인가?

## Interview Drills

### Q1
읽기 95%, 쓰기 5%인 서비스에서 DB CPU가 80%를 넘었다. 어떤 순서로 병목을 확인하고 어떤 확장 전략을 검토하겠는가?

### Q2
사용자가 프로필을 수정한 직후 이전 값이 보인다는 신고가 들어왔다. Read Replica를 쓰고 있다면 무엇을 의심하겠는가?

### Q3
App instance를 5대에서 30대로 늘렸더니 DB connection error가 증가했다. 왜 이런 일이 생길 수 있는가?

### Q4
`트래픽이 크므로 처음부터 Sharding하겠습니다`라는 답변의 문제점을 설명하라.

### Q5
좋아요 수를 한 Row에 계속 `UPDATE`하는 구조가 고부하에서 병목이 되는 이유와 대안을 설명하라.

## 60-second Test

자료 없이 60초 안에 다음을 포함해 설명한다.

- Read vs Write pattern
- Read Replica
- Replica Lag
- Cache
- Write hotspot
- Sharding trade-off
- DB Connection capacity
