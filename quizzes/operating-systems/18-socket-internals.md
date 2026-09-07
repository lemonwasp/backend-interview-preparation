# 18. Socket Internals — 이해도 확인

상태: **답변 대기**

## A. 파인만 확인 질문

### Q1
Socket을 쉬운 말로 설명해보세요.

### Q2
Listening Socket과 Connected Socket은 무엇이 다른가요?

### Q3
`socket → bind → listen → accept` 흐름을 설명해보세요.

### Q4
Socket Send Buffer와 Receive Buffer는 어디에 있고 무엇을 하나요?

## B. 동작 질문

### Q5
`send()`가 성공했다고 상대방 애플리케이션이 데이터를 읽었다고 볼 수 없는 이유는 무엇인가요?

### Q6
Blocking Socket의 Send Buffer가 가득 차면 어떤 일이 일어날 수 있나요?

### Q7
Non-blocking Socket에서는 같은 상황이 어떻게 다르게 보이나요?

### Q8
accept queue가 가득 차면 서버에 어떤 현상이 생길 수 있나요?

## C. 백엔드 연결 질문

### Q9
TCP Connection 하나가 소비하는 자원을 네 가지 이상 말해보세요.

### Q10
Connection 10만 개를 처리할 때 Thread 수만 보면 안 되는 이유는 무엇인가요?

### Q11
C# `ReceiveAsync`가 connection마다 전용 Thread를 하나씩 만든다고 보면 안 되는 이유는 무엇인가요?

### Q12
Thread-per-connection과 event-driven I/O 모델의 핵심 trade-off를 설명해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 구조 | socket을 Kernel network endpoint로 이해하는가 |
| 서버 흐름 | bind/listen/accept를 구분하는가 |
| Buffer | send/receive buffer의 위치와 의미를 설명하는가 |
| 확장성 | FD/buffer/TCP state/thread 비용을 연결하는가 |
| Async | async I/O와 thread-per-connection을 구분하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
