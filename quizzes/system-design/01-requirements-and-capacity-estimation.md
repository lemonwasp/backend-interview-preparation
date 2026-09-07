# 01. Requirements and Capacity Estimation — Interview Drills

상태: **답변 대기**

## Questions

1. Functional Requirement와 Non-functional Requirement의 차이를 예시와 함께 설명하세요.
2. System Design 면접에서 기술 선택보다 요구사항 확인을 먼저 해야 하는 이유는 무엇인가요?
3. DAU 100만 명, 사용자당 하루 20회 요청이라면 평균 RPS를 대략 계산해보세요.
4. 평균 RPS만으로 서버 용량을 정하면 위험한 이유는 무엇인가요?
5. Read/Write ratio를 알아야 하는 이유를 설명하세요.
6. Storage growth를 추정하면 어떤 설계 결정에 도움이 되나요?
7. 100KB 응답을 초당 1,000개 내려준다면 대략 어느 정도의 bandwidth가 필요한가요?
8. Latency Budget이란 무엇이며 downstream마다 전체 timeout을 그대로 주면 왜 안 되나요?
9. Availability를 99.9%에서 99.99%로 올리는 것이 왜 단순 숫자 차이가 아닌가요?
10. 모든 데이터에 Strong Consistency가 필요하지 않은 이유를 예시와 함께 설명하세요.
11. System Design에서 estimation이 정확한 예측이 아니라도 유용한 이유는 무엇인가요?
12. 이미지 변환 API를 설계한다면 가장 먼저 어떤 숫자들을 확인하겠습니까?
13. 면접관이 “DAU 1천만”만 줬을 때 추가로 어떤 질문을 하겠습니까?

## Evaluation Criteria

- 기술 이름부터 시작하지 않고 요구사항에서 출발하는가
- 평균과 peak를 구분하는가
- RPS/storage/bandwidth를 대략 계산할 수 있는가
- Latency/Availability/Consistency trade-off를 설명하는가
- 숫자를 architecture decision과 연결하는가
- 60초 안에 설계 시작 절차를 구조적으로 설명할 수 있는가
