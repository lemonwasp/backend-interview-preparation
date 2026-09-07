# 08. Concurrency vs Parallelism — 이해도 확인

상태: **답변 대기**

## A. 개념 구분

### Q1
Concurrency와 Parallelism의 차이를 요리사 비유 없이 자신의 말로 설명해보세요.

### Q2
Single Core에서도 Concurrency가 가능한 이유는 무엇인가요?

### Q3
Single Core에서 CPU-bound 작업 두 개가 실제로 동시에 실행될 수 없는 이유는 무엇인가요?

### Q4
Multi Core가 있어도 프로그램이 자동으로 빨라지지 않는 이유를 세 가지 이상 말해보세요.

## B. CPU-bound / I/O-bound

### Q5
CPU-bound 작업에서는 왜 Parallelism을 고려하고, I/O-bound 작업에서는 왜 Concurrency가 중요할까요?

### Q6
100개의 Thread를 4 Core CPU에서 실행한다고 해서 100-way Parallelism이 되지 않는 이유는 무엇인가요?

### Q7
async HTTP 서버가 높은 Concurrency를 가지면서도 모든 요청이 동시에 CPU에서 실행되는 것은 아닌 이유를 설명해보세요.

## C. C# 연결

### Q8
C# `Task`와 Thread는 같은 개념인가요? 아니라면 어떻게 다른가요?

### Q9
`await httpClient.GetAsync()`와 `Task.Run(HeavyCalculation)`의 실행 성격 차이를 설명해보세요.

### Q10
`async`와 Parallelism을 동일시하면 안 되는 이유는 무엇인가요?

## D. Shared State

### Q11
두 Thread가 같은 `counter`에 `counter++`를 수행하면 문제가 생길 수 있는 이유를 단계 수준에서 설명해보세요.

### Q12
Shared State를 줄이면 동시성 코드가 단순해지는 이유는 무엇인가요?

### Q13
TIFF-to-PDF 페이지 변환을 병렬화하기 전에 확인해야 할 조건을 네 가지 이상 말해보세요.

## E. 기술면접 질문

### Q14
“Concurrency와 Parallelism의 차이가 무엇인가요?”에 60초 이내로 답해보세요.

### Q15
“Thread를 100개 만들면 100배 빨라지나요?”라는 꼬리 질문에 답해보세요.

### Q16
“Amdahl's Law가 병렬 프로그래밍에서 주는 핵심 직관은 무엇인가요?”에 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 개념 구분 | concurrency와 parallelism을 분리하는가 |
| 하드웨어 | Core 수와 실제 병렬 실행을 연결하는가 |
| workload | CPU-bound/I/O-bound 전략을 구분하는가 |
| C# 정확성 | Task/Thread/async를 혼동하지 않는가 |
| 실무 연결 | shared state와 측정 기반 병렬화를 설명하는가 |

## 평가 결과

답변 제출 후 업데이트 예정입니다.

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
