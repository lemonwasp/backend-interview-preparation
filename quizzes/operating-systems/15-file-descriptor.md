# 15. File Descriptor — 이해도 확인

상태: **답변 대기**

## A. 파인만 확인 질문

### Q1
File Descriptor를 컴퓨터를 모르는 사람에게 설명해보세요.

### Q2
FD 자체가 파일 데이터가 아닌 이유는 무엇인가요?

### Q3
Process A의 FD 3과 Process B의 FD 3이 같은 파일을 의미하나요?

### Q4
왜 Socket도 Unix 계열에서 FD로 다룰 수 있나요?

## B. 실무 연결 질문

### Q5
FD Leak이 발생하면 백엔드 서버에서 어떤 장애가 생길 수 있나요?

### Q6
DB Connection Pool을 지나치게 크게 잡으면 FD 관점에서 어떤 문제가 생길 수 있나요?

### Q7
C#에서 `using` 또는 `Dispose()`가 OS 자원 관리와 연결되는 이유를 설명해보세요.

### Q8
`Too many open files`가 발생했을 때 어떤 종류의 자원 누수를 의심할 수 있나요?

## C. 기술면접 질문

### Q9
“File Descriptor란 무엇인가요?”에 60초 이내로 답해보세요.

### Q10
“파일과 Socket은 전혀 다른데 왜 같은 FD abstraction으로 다룰 수 있나요?”에 답해보세요.

### Q11
`fork()`와 FD 상속이 자원 정리를 복잡하게 만들 수 있는 이유는 무엇인가요?

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 정의 | FD를 Process-local integer handle로 설명하는가 |
| 구조 | FD table과 Kernel resource를 구분하는가 |
| 실무 | FD leak을 실제 장애와 연결하는가 |
| 추상화 | file/socket/pipe의 공통 I/O interface를 이해하는가 |
| 자원관리 | Dispose/close의 필요성을 설명하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
