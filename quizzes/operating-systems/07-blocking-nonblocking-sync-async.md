# 07. Blocking / Non-blocking / Sync / Async — 이해도 확인

상태: **답변 대기**

## A. 개념 구분

### Q1
Blocking과 Non-blocking을 무엇을 기준으로 구분하나요?

### Q2
Synchronous와 Asynchronous를 무엇을 기준으로 구분하나요?

### Q3
왜 `Blocking = Sync`, `Non-blocking = Async`라고 단순하게 외우면 안 되나요?

### Q4
Blocking I/O를 기다리는 동안 CPU 전체가 멈추는 것이 아닌 이유를 설명해보세요.

### Q5
Non-blocking 호출을 Busy Polling으로 구현하면 어떤 문제가 생길 수 있나요?

## B. 백엔드 연결

### Q6
요청 하나당 Thread 하나가 Blocking DB 호출을 기다리는 서버에서 동시 요청이 증가하면 어떤 문제가 생길 수 있나요?

### Q7
Async I/O가 실제 네트워크 지연 시간을 줄이지 않아도 서버 확장성에 도움이 되는 이유는 무엇인가요?

### Q8
Thread Pool 고갈이 어떤 상황에서 발생하는지 설명해보세요.

### Q9
Linux `epoll`이나 Windows IOCP 같은 메커니즘이 왜 많은 연결을 적은 Thread로 처리하는 데 도움이 되나요?

## C. C# 연결

### Q10
C# `await httpClient.GetAsync(...)`가 새 Thread를 하나 만드는 코드가 아닌 이유를 설명해보세요.

### Q11
CPU-bound 함수를 단순히 `async`로 선언한다고 빨라지지 않는 이유는 무엇인가요?

### Q12
`Task`, Thread, async I/O의 관계를 가능한 정확하게 설명해보세요.

## D. 기술면접 질문

### Q13
“Blocking과 Non-blocking의 차이가 무엇인가요?”에 30초 이내로 답해보세요.

### Q14
“Sync와 Async는 Blocking/Non-blocking과 같은 개념 아닌가요?”라는 꼬리 질문에 답해보세요.

### Q15
“async/await를 쓰면 서버가 항상 빨라지나요?”에 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 축 분리 | Blocking 축과 Sync/Async 축을 분리하는가 |
| Thread 관점 | I/O 대기와 Thread 상태를 설명하는가 |
| 확장성 | Thread Pool/Context Switch와 연결하는가 |
| C# 정확성 | await = 새 Thread라는 오해가 없는가 |
| 성능 | latency와 resource efficiency를 구분하는가 |

## 평가 결과

답변 제출 후 업데이트 예정입니다.

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
