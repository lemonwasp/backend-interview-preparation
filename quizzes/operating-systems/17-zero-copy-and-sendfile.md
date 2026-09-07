# 17. Zero-copy와 sendfile — 이해도 확인

상태: **답변 대기**

## A. 파인만 확인 질문

### Q1
Zero-copy를 쉬운 말로 설명해보세요.

### Q2
일반적인 file read → socket send 경로에서 어떤 데이터 복사가 생길 수 있나요?

### Q3
`sendfile`은 무엇을 줄이기 위한 API인가요?

### Q4
Zero-copy가 물리적으로 복사가 정말 0번이라는 뜻이 아닌 이유는 무엇인가요?

## B. 성능 질문

### Q5
불필요한 메모리 복사가 CPU 성능에 부담을 주는 이유를 두 가지 말해보세요.

### Q6
정적 파일 서버나 CDN에서 zero-copy가 특히 유리한 이유는 무엇인가요?

### Q7
압축이나 암호화가 필요한 경우 단순 `sendfile` 경로가 어려워질 수 있는 이유는 무엇인가요?

### Q8
TLS가 있다고 해서 zero-copy 최적화가 절대 불가능하다고 단정하면 안 되는 이유는 무엇인가요?

## C. 실무 연결 질문

### Q9
TIFF→PDF에서 temp file을 `MemoryStream`으로 바꾼 사례와 zero-copy의 공통된 사고방식은 무엇인가요?

### Q10
“MemoryStream을 썼으니 zero-copy입니다”라는 표현이 부정확한 이유는 무엇인가요?

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 정의 | 불필요한 copy 감소가 핵심임을 설명하는가 |
| 경로 | User/Kernel buffer 흐름을 이해하는가 |
| 정확성 | zero-copy를 literal zero copies로 오해하지 않는가 |
| 실무 | static file/CDN/proxy와 연결하는가 |
| 최적화 | TIFF 사례와 공통 원칙을 구분해서 설명하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
