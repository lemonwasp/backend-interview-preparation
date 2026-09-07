# 13. Graceful Degradation — Knowledge Check

상태: 답변 대기

## 기본 질문

1. Graceful Degradation이란 무엇인가요?
2. Critical / Important / Optional 기능을 나누는 이유는 무엇인가요?
3. Fallback과 Retry는 어떻게 다르나요?
4. stale cache를 사용해도 되는 데이터와 사용하면 위험한 데이터의 예를 들어보세요.
5. Feature Shedding과 Load Shedding의 차이는 무엇인가요?
6. Circuit Breaker와 Fallback은 어떻게 함께 사용되나요?
7. 무제한 Retry가 장애를 더 키울 수 있는 이유는 무엇인가요?
8. Queue로 비동기화하는 것이 degradation 전략이 될 수 있는 이유는 무엇인가요?

## 꼬리 질문

9. 추천 서비스 장애 때문에 상품 상세 API 전체가 500을 반환합니다. 어떻게 바꾸겠습니까?
10. Cache 원본 DB가 죽었습니다. stale cache를 언제까지 제공할지 어떤 기준으로 정하겠습니까?
11. 결제 상태 조회에 stale cache를 쓰는 것이 위험한 이유는 무엇인가요?
12. 시스템이 과부하 상태일 때 모든 요청을 queue에 쌓는 것이 왜 더 위험할 수 있나요?
13. Partial Response를 설계할 때 API 계약에서 무엇을 명확히 해야 하나요?
14. Graceful Degradation이 사용자 UX와 연결되는 이유는 무엇인가요?

## 시나리오

15. 트래픽 급증으로 DB가 포화되기 시작했습니다. 추천, 통계, 결제 기능이 모두 같은 DB를 사용합니다. 어떤 기능부터 줄이겠습니까?
16. 외부 배송 API가 30초씩 timeout 납니다. 주문 API의 p99가 폭증했습니다. 어떤 보호 장치를 적용하겠습니까?
17. 뉴스 서비스의 원본 API가 장애입니다. 10분 전 cache가 있습니다. 어떻게 판단하겠습니까?

## 60초 면접 답변

18. "Dependency 장애 시 서비스 전체를 살리는 방법을 설명해보세요"에 60초 안에 답해보세요.

## 평가 기준

- Critical path와 optional feature를 구분하는가
- timeout/retry/circuit breaker/fallback의 역할을 구분하는가
- stale data의 안전성을 데이터 특성에 따라 판단하는가
- load shedding을 단순 실패가 아니라 system protection으로 이해하는가
- 사용자에게 중간 상태를 어떻게 노출할지도 고려하는가
