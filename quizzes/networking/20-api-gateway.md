# 20. API Gateway — 이해도 확인 문제

1. API Gateway의 핵심 역할은 무엇입니까?
2. Reverse Proxy와 API Gateway의 차이를 설명하세요.
3. Load Balancer와 API Gateway의 책임 차이는 무엇입니까?
4. Authentication을 Gateway에서 수행해도 Backend Authorization이 필요한 이유는 무엇입니까?
5. Rate Limiting을 Gateway에 두기 좋은 이유는 무엇입니까?
6. 여러 Gateway instance에서 Rate Limit counter를 관리할 때 어떤 문제가 생길 수 있습니까?
7. Request Aggregation의 장점과 단점은 무엇입니까?
8. Gateway가 Bottleneck 또는 SPOF가 될 수 있는 이유를 설명하세요.
9. Gateway 자체를 어떻게 고가용성으로 구성할 수 있습니까?
10. Client Retry × Gateway Retry × Service Retry가 왜 위험합니까?
11. Gateway에서 수집하기 좋은 observability 지표를 최소 5개 말하세요.
12. API Gateway와 Service Mesh의 기본적인 traffic 방향 차이를 설명하세요.
13. Gateway에 business logic을 과도하게 넣으면 어떤 문제가 생깁니까?
14. 60초 안에 API Gateway의 역할과 trade-off를 설명해 보세요.

## 평가 기준

- Gateway / Reverse Proxy / Load Balancer를 구분하는가
- 인증과 도메인 권한 검사를 분리해서 설명하는가
- Rate Limit, Retry, HA, observability를 운영 관점으로 연결하는가
- Gateway를 단일 진입점이라는 이유만으로 단일 인스턴스로 오해하지 않는가

## 상태

답변 대기
