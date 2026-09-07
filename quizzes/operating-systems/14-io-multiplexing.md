# 14. I/O Multiplexing — 이해도 확인

상태: **답변 대기**

## A. 핵심 개념

### Q1
Connection마다 Thread 하나를 붙이는 구조가 대규모 서버에서 비효율적일 수 있는 이유는 무엇인가요?

### Q2
I/O Multiplexing의 핵심 아이디어를 한 문장으로 설명해보세요.

### Q3
`select`, `poll`, `epoll`의 차이를 큰 그림에서 설명해보세요.

### Q4
`epoll`이 많은 Connection에서 `select/poll`보다 유리할 수 있는 이유는 무엇인가요?

## B. 플랫폼 모델

### Q5
Readiness 기반 모델과 Completion 기반 모델의 차이를 설명해보세요.

### Q6
`epoll`과 IOCP를 완전히 같은 모델이라고 보면 안 되는 이유는 무엇인가요?

### Q7
Level-triggered와 Edge-triggered의 차이를 설명해보세요.

### Q8
Edge-triggered 방식에서 데이터를 충분히 읽지 않으면 문제가 생길 수 있는 이유는 무엇인가요?

## C. Runtime과 백엔드

### Q9
Event Loop와 I/O Multiplexing은 어떻게 연결되나요?

### Q10
ASP.NET Core의 `await httpClient.GetAsync(...)`가 요청 대기 동안 Thread를 효율적으로 사용할 수 있는 이유를 OS 관점에서 설명해보세요.

### Q11
`async/await`가 곧 `epoll` 자체를 의미하지 않는 이유는 무엇인가요?

### Q12
I/O Multiplexing이 CPU Parallelism과 다른 이유를 설명해보세요.

### Q13
Event Loop에서 Image Processing 같은 CPU-heavy 작업을 오래 수행하면 어떤 문제가 생기나요?

## D. 기술면접

### Q14
“I/O Multiplexing이 무엇인가요?”에 60초 이내로 답해보세요.

### Q15
“그럼 Thread Pool은 필요 없나요?”라는 꼬리 질문에 답해보세요.

### Q16
“10,000개의 Connection이면 Thread도 10,000개 필요한가요?”라는 질문에 답해보세요.

## 평가 기준

- Thread-per-connection의 비용을 설명하는가
- select/poll/epoll의 큰 차이를 설명하는가
- readiness와 completion을 구분하는가
- Event Loop / async-await / OS I/O 계층을 연결하는가
- I/O Concurrency와 CPU Parallelism을 구분하는가
