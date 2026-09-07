# 16. Interrupt와 DMA — 이해도 확인

상태: **답변 대기**

## A. 파인만 확인 질문

### Q1
Polling과 Interrupt의 차이를 설명해보세요.

### Q2
Interrupt가 없으면 CPU가 어떤 비효율을 겪을 수 있나요?

### Q3
DMA는 왜 필요한가요?

### Q4
DMA를 사용하면 CPU는 무엇을 하지 않아도 되나요?

## B. 흐름 문제

### Q5
NIC에 packet이 도착한 뒤 애플리케이션이 데이터를 읽기까지의 큰 흐름을 설명해보세요.

### Q6
SSD read 요청이 완료되었을 때 waiting Thread가 다시 실행되기까지의 흐름을 설명해보세요.

### Q7
Interrupt가 항상 Polling보다 좋은 방식이라고 할 수 없는 이유는 무엇인가요?

### Q8
Interrupt Storm이 무엇인지 설명해보세요.

## C. 백엔드 연결 질문

### Q9
CPU 사용률이 낮아도 서버가 느릴 수 있는 이유를 I/O 관점에서 설명해보세요.

### Q10
DMA를 이해하면 zero-copy를 이해하기 쉬워지는 이유는 무엇인가요?

### Q11
“async I/O는 CPU가 디스크 데이터를 직접 복사하지 않아서 가능한 것인가요?”라는 질문에 과장 없이 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| Interrupt | event notification 메커니즘으로 설명하는가 |
| Polling | CPU가 직접 상태를 반복 확인함을 이해하는가 |
| DMA | CPU 대신 장치↔메모리 데이터 전송을 수행함을 설명하는가 |
| 흐름 | NIC/Disk → Kernel → Application 흐름을 연결하는가 |
| Trade-off | Interrupt와 Polling을 절대적으로 비교하지 않는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
