# 11. Service Boundary / Ownership — Knowledge Check

상태: 답변 대기

## 기본 질문

1. Service Boundary를 나눌 때 기술 계층보다 Business Capability를 보는 이유는 무엇인가요?
2. Data Ownership과 Source of Truth는 무엇을 의미하나요?
3. 다른 서비스가 내 서비스의 DB 테이블을 직접 수정하면 어떤 문제가 생기나요?
4. Shared Database의 장점과 단점은 무엇인가요?
5. Database per Service가 주는 독립성과 그 비용을 설명해보세요.
6. 서비스 간 동기 API와 비동기 Event를 각각 언제 선택하겠습니까?
7. 강한 Transaction이 자주 필요한 데이터가 서비스 경계에 어떤 영향을 주나요?
8. Modular Monolith가 Microservice보다 더 좋은 출발점이 될 수 있는 경우는 언제인가요?

## 꼬리 질문

9. Order, Payment, Inventory를 무조건 각각 독립 서비스로 나누면 어떤 새로운 문제가 생길까요?
10. Event-driven architecture가 서비스 간 결합을 완전히 제거하지 못하는 이유는 무엇인가요?
11. 팀 규모가 4명인데 서비스가 20개라면 어떤 운영 비용을 예상하나요?
12. 한 서비스만 트래픽이 100배 증가한다면 서비스 분리를 검토할 근거가 될 수 있는 이유는 무엇인가요?
13. 서비스 A가 서비스 B를 동기로 호출하고 B가 C를 호출할 때 어떤 failure propagation이 생길 수 있나요?
14. 서비스 분리 후 Cross-service JOIN이 필요해졌다면 어떤 대안을 검토할 수 있나요?

## 시나리오

15. 주문 생성 시 주문 헤더와 주문 라인을 반드시 함께 저장해야 합니다. 이를 각각 다른 Microservice로 나누자는 제안이 있습니다. 어떻게 판단하겠습니까?
16. Payment Service가 주문 상태를 직접 UPDATE하고 있습니다. 어떤 문제가 있고 어떻게 바꾸겠습니까?
17. 처음부터 Microservice로 갈지 Modular Monolith로 갈지 결정해야 합니다. 판단 기준을 설명하세요.

## 60초 면접 답변

18. "Microservice의 서비스 경계는 어떻게 정합니까?"에 60초 안에 답해보세요.

## 평가 기준

- Business Capability / Data Ownership / Transaction Boundary를 연결해서 설명하는가
- Shared DB와 Database per Service를 흑백논리 없이 trade-off로 설명하는가
- Sync API와 Async Event의 장애/일관성 비용을 설명하는가
- Microservice 자체를 목표로 삼지 않고 조직·트래픽·배포 요구사항을 근거로 판단하는가
