# 15. CDN과 HTTP Cache — 이해도 확인

상태: **답변 대기**

## A. 핵심 개념

### Q1
CDN이 Latency와 Origin 부하를 줄이는 원리를 설명하세요.

### Q2
Browser Cache와 CDN Cache의 차이는 무엇인가요?

### Q3
HTTP Cache에서 Freshness와 Validation의 차이를 설명하세요.

### Q4
`Cache-Control: max-age=3600`은 무엇을 의미하나요?

## B. Header 이해

### Q5
`no-cache`와 `no-store`의 차이를 설명하세요.

### Q6
`private`와 `public`은 어떤 상황에서 중요하나요?

### Q7
ETag와 `If-None-Match`가 어떻게 동작하는지 설명하세요.

### Q8
304 Not Modified 응답이 성능에 도움이 되는 이유는 무엇인가요?

## C. 실무·보안

### Q9
Cache Key를 잘못 설계하면 어떤 보안 문제가 생길 수 있나요?

### Q10
인증된 사용자별 API Response를 CDN에 Cache할 때 무엇을 주의해야 하나요?

### Q11
Cache Invalidation이 어려운 이유와 대표적인 대응 방법을 설명하세요.

### Q12
Content Hash가 포함된 Static Asset URL이 유리한 이유는 무엇인가요?

### Q13
Cache Stampede가 무엇이며 어떻게 완화할 수 있나요?

## D. 기술면접

### Q14
“CDN이 무엇이고 왜 사용하나요?”에 60초 이내로 답하세요.

### Q15
“`no-cache`는 캐시하지 말라는 뜻인가요?”라는 꼬리 질문에 답하세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 계층 | Browser / Shared / CDN Cache를 구분하는가 |
| HTTP | Cache-Control / ETag / 304 흐름을 설명하는가 |
| 보안 | Cache Key와 사용자별 데이터 위험을 이해하는가 |
| 운영 | Invalidation / Stampede 문제를 설명하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
