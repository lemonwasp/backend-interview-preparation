# 13. Forward Proxy와 Reverse Proxy — 이해도 확인

상태: **답변 대기**

## A. 핵심 개념

### Q1
Forward Proxy와 Reverse Proxy의 가장 중요한 차이는 무엇인가요?

### Q2
Forward Proxy의 대표적인 사용 사례를 세 가지 말해보세요.

### Q3
Reverse Proxy가 수행할 수 있는 기능을 네 가지 이상 말해보세요.

### Q4
Reverse Proxy 뒤에 여러 Backend가 있어도 Client가 이를 몰라도 되는 이유는 무엇인가요?

## B. 실무 연결

### Q5
Nginx가 TLS를 종료하고 ASP.NET Core Backend로 HTTP를 전달하는 구조의 장단점을 설명하세요.

### Q6
Reverse Proxy 뒤에서 실제 Client IP가 필요한 경우 어떻게 전달할 수 있나요?

### Q7
`X-Forwarded-For`를 무조건 신뢰하면 안 되는 이유는 무엇인가요?

### Q8
Proxy 계층이 늘어날수록 생길 수 있는 성능·운영 비용을 설명하세요.

## C. 기술면접

### Q9
“Forward Proxy와 Reverse Proxy의 차이를 설명해주세요.”에 60초 이내로 답하세요.

### Q10
“Reverse Proxy와 Load Balancer는 같은 건가요?”라는 꼬리 질문에 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 핵심 구분 | Client 대신 / Server 대신을 구분하는가 |
| 실무 | TLS, Routing, Cache, LB와 연결하는가 |
| 보안 | Forwarded Header 신뢰 경계를 설명하는가 |
| 운영 | Proxy 추가의 비용도 설명하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
