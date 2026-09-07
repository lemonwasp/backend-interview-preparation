# 05. Context Switching — 이해도 확인

상태: **답변 대기**

문서를 보지 않고 자신의 말로 답합니다.

## A. 파인만 확인 질문

### Q1

Context Switch가 무엇인지 설명해보세요.

### Q2

Context Switch 때 왜 현재 Thread의 상태를 저장해야 하나요?

### Q3

Program Counter, Register, Stack Pointer는 각각 왜 중요한가요?

### Q4

운영체제가 실행 중인 Thread를 바꾸는 대표적인 상황을 세 가지 말해보세요.

## B. 비용과 성능

### Q5

Context Switch에는 왜 비용이 발생하나요?

### Q6

CPU Cache와 TLB 관점에서 Context Switch가 성능에 어떤 영향을 줄 수 있나요?

### Q7

같은 Process 안의 Thread Switch와 서로 다른 Process 사이 Switch가 비용 면에서 다를 수 있는 이유는 무엇인가요?

## C. 개념 구분

### Q8

Mode Switch와 Context Switch의 차이를 설명해보세요.

### Q9

System Call이 발생해도 Context Switch가 일어나지 않을 수 있는 예를 하나 들어보세요.

### Q10

I/O 대기 때문에 현재 Thread가 Block되면 Context Switch가 발생할 수 있는 이유를 설명해보세요.

## D. 백엔드 연결

### Q11

Runnable Thread가 지나치게 많으면 백엔드 서버에 어떤 문제가 생길 수 있나요?

### Q12

Thread Pool이 Context Switch와 자원 사용을 제어하는 데 어떻게 도움이 되나요?

### Q13

C# `async/await`가 I/O-bound 서버에서 Thread 자원을 효율적으로 사용할 수 있게 하는 이유를 설명해보세요.

### Q14

“async/await를 쓰면 Context Switch가 사라진다”는 말이 왜 틀렸나요?

### Q15

TIFF-to-PDF 변환을 병렬화할 때 CPU Core 수를 무시하고 Thread만 늘리면 어떤 문제가 생길 수 있나요?

## E. 기술면접 질문

### Q16

“Context Switch가 무엇이고 왜 비싼가요?”에 60초 이내로 답해보세요.

### Q17

“Context Switch는 나쁜 것인데 왜 운영체제가 사용하나요?”라는 꼬리 질문에 답해보세요.

### Q18

CPU-bound와 I/O-bound 작업에서 Thread/Async 전략이 왜 달라지는지 설명해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 정의 | 실행 상태 저장/복원과 실행 주체 전환을 설명하는가 |
| 비용 | Scheduler, Register, Cache/TLB 비용을 설명하는가 |
| 구분 | Mode Switch와 Context Switch를 구분하는가 |
| 동시성 | Thread 수 증가가 무조건 성능 향상이 아님을 설명하는가 |
| 백엔드 연결 | Thread Pool, async/await, I/O-bound에 연결하는가 |
| 실무 연결 | TIFF 변환의 CPU-bound 병렬화 위험을 설명하는가 |

## 평가 결과

답변 제출 후 업데이트 예정입니다.

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
