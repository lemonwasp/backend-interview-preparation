# 14. Failure Mode Reasoning — Knowledge Check

상태: 답변 대기

## 기본 질문

1. Failure Mode Reasoning이란 무엇인가요?
2. Failure Domain은 무엇이고 왜 중요한가요?
3. Partial Failure가 분산 시스템을 어렵게 만드는 이유는 무엇인가요?
4. 느린 dependency도 failure로 봐야 하는 이유는 무엇인가요?
5. Retry Storm은 어떻게 발생하나요?
6. Queue 장애에서 Queue Depth만 보면 부족한 이유는 무엇인가요?
7. Cache 장애가 DB 장애로 전파될 수 있는 과정을 설명해보세요.
8. DB write timeout 후 무조건 retry하면 위험한 이유는 무엇인가요?
9. Blast Radius란 무엇이고 어떻게 줄일 수 있나요?
10. SPOF를 모두 제거하지 않는 이유는 무엇인가요?

## 꼬리 질문

11. 같은 Availability Zone에 replica 3개를 두면 어떤 failure에 취약한가요?
12. 전체 요청 deadline이 1초인데 downstream timeout이 5초라면 어떤 문제가 생기나요?
13. Consumer backlog가 증가했을 때 consumer 수를 무조건 늘리면 왜 위험한가요?
14. Cache와 DB가 동시에 실패할 수 있는 공통 원인은 무엇이 있을까요?
15. 장애 발생 시 "최근 무엇이 바뀌었는가"를 먼저 확인하는 이유는 무엇인가요?
16. Canary Deployment와 Feature Flag가 blast radius를 줄이는 데 어떻게 도움이 되나요?

## 시나리오

17. 외부 API가 평소 100ms에서 10초로 느려졌습니다. 시스템 전체 p99가 급등하고 ThreadPool queue가 증가합니다. 원인과 대응을 설명하세요.
18. Cache Cluster가 내려가자 DB CPU가 100%가 됐습니다. 즉시 완화와 근본 대책을 설명하세요.
19. Queue backlog가 1시간치 쌓였다가 consumer가 복구됐습니다. 어떻게 drain하겠습니까?
20. 고객이 결제 timeout을 받았지만 실제 결제 여부를 알 수 없습니다. 어떤 설계가 필요합니까?

## 60초 면접 답변

21. "분산 시스템 장애를 어떻게 분석하고 설계 단계에서 대비합니까?"에 60초 안에 답해보세요.

## 평가 기준

- down뿐 아니라 slow/partial failure를 포함하는가
- failure propagation과 cascading failure를 설명하는가
- timeout/retry/idempotency를 연결하는가
- blast radius와 failure domain을 구분하는가
- 즉시 완화와 데이터 정합성/재처리/재발 방지를 모두 고려하는가
