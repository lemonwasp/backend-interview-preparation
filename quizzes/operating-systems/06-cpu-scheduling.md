# 06. CPU Scheduling — 이해도 확인

상태: **답변 대기**

## A. 파인만 확인 질문

### Q1
CPU Scheduling이 왜 필요한지 CPU Core 하나를 예로 들어 설명해보세요.

### Q2
Ready, Running, Waiting 상태를 각각 설명하고 상태가 어떻게 바뀌는지 예를 들어보세요.

### Q3
Preemptive와 Non-preemptive Scheduling의 차이는 무엇인가요?

### Q4
FCFS에서 Convoy Effect가 왜 발생하나요?

### Q5
SJF가 평균 Waiting Time을 줄이는 데 유리한데도 실제 시스템에서 그대로 쓰기 어려운 이유는 무엇인가요?

### Q6
Round Robin의 Time Quantum이 너무 크거나 너무 작으면 각각 어떤 문제가 생기나요?

### Q7
Priority Scheduling의 Starvation과 Aging을 설명해보세요.

## B. 지표 구분

### Q8
Throughput, Turnaround Time, Waiting Time, Response Time의 차이를 설명해보세요.

### Q9
웹 서비스에서 평균 Turnaround Time보다 Response Time이 특히 중요할 수 있는 이유는 무엇인가요?

## C. 백엔드 연결

### Q10
CPU-bound 작업에 CPU Core 수보다 훨씬 많은 Thread를 만들면 왜 성능이 나빠질 수 있나요?

### Q11
I/O-bound 작업은 왜 CPU-bound 작업보다 높은 동시성이 유리할 수 있나요?

### Q12
Thread Pool이 CPU Scheduling 관점에서 어떤 문제를 완화하나요?

### Q13
4 Core 서버에서 30개의 이미지 변환 작업을 모두 CPU-bound Thread로 동시에 실행하는 전략의 문제점을 설명해보세요.

## D. 기술면접 질문

### Q14
“CPU Scheduling이 무엇인가요?”에 60초 이내로 답해보세요.

### Q15
“Thread를 많이 만들면 서버 처리량도 계속 늘어나지 않나요?”라는 꼬리 질문에 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 상태 모델 | Ready/Running/Waiting을 구분하는가 |
| 정책 이해 | FCFS/SJF/RR/Priority의 trade-off를 설명하는가 |
| 지표 | Response/Waiting/Turnaround/Throughput을 혼동하지 않는가 |
| 실무 연결 | CPU-bound/I/O-bound와 Thread 수를 연결하는가 |
| 비용 인식 | Context Switch와 Cache 손실을 언급하는가 |

## 평가 결과

답변 제출 후 업데이트 예정입니다.

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
