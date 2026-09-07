# 12. Certificate와 PKI — 이해도 확인

상태: **답변 대기**

## A. 핵심 개념

### Q1
Certificate는 무엇을 증명하는 문서인가요?

### Q2
Certificate 안에 Private Key가 들어있지 않은 이유를 설명하세요.

### Q3
Root CA / Intermediate CA / Server Certificate의 관계를 설명하세요.

### Q4
PKI가 필요한 이유는 무엇인가요?

## B. 검증 흐름

### Q5
브라우저가 Server Certificate를 검증할 때 확인하는 항목을 네 가지 이상 말해보세요.

### Q6
SAN과 Domain Name은 어떤 관계인가요?

### Q7
Root CA가 Self-signed여도 신뢰될 수 있는 이유는 무엇인가요?

## C. 실무 연결

### Q8
Intermediate Certificate 누락이 일부 Client에서만 장애를 일으킬 수 있는 이유를 설명하세요.

### Q9
Certificate Expiration을 예방하기 위한 운영 방법을 말해보세요.

### Q10
서버 시간이 크게 잘못되어 있다면 왜 TLS 검증이 실패할 수 있나요?

## D. 기술면접

### Q11
“브라우저는 서버 인증서를 어떻게 신뢰하나요?”에 60초 이내로 답하세요.

### Q12
“Root CA는 누가 인증하나요?”라는 꼬리 질문에 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 인증서 | 공개키와 신원의 연결을 설명하는가 |
| Chain | Leaf → Intermediate → Root를 설명하는가 |
| Trust | Trust Store가 신뢰의 시작점임을 설명하는가 |
| 운영 | 만료·Chain 누락·시간 오류를 장애와 연결하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
