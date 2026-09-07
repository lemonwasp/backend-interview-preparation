# 07. DNS — 이해도 확인

상태: **답변 대기**

## A. 기본 개념

### Q1
DNS가 왜 필요한지 설명하세요.

### Q2
Recursive Resolver와 Authoritative DNS Server의 역할 차이는 무엇인가요?

### Q3
Root → TLD → Authoritative DNS 흐름을 설명해보세요.

### Q4
A, AAAA, CNAME Record의 차이는 무엇인가요?

## B. Cache와 운영

### Q5
DNS TTL이 길면 어떤 장점과 단점이 있나요?

### Q6
서비스 IP를 변경할 예정이라면 TTL을 왜 신경 써야 하나요?

### Q7
애플리케이션 서버가 정상인데도 DNS 문제 때문에 서비스 장애가 발생할 수 있는 이유는 무엇인가요?

### Q8
“DNS는 항상 UDP를 사용한다”가 왜 틀린 설명인가요?

## C. 기술면접

### Q9
브라우저에 URL을 입력한 뒤 DNS 관점에서 어떤 일이 일어나는지 60초 이내로 설명하세요.

### Q10
DNS Cache가 성능을 높이는 동시에 장애 복구를 늦출 수 있는 이유를 설명하세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 계층 구조 | Root/TLD/Authoritative 관계를 이해하는가 |
| Resolver | 재귀 질의를 대신 수행하는 역할을 설명하는가 |
| Cache | TTL의 성능/운영 trade-off를 설명하는가 |
| 실무 | 장애·Failover·Service discovery와 연결하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
