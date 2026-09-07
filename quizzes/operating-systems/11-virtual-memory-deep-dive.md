# 11. Virtual Memory 심화 — 이해도 확인

상태: **답변 대기**

## A. 핵심 개념

### Q1
Virtual Address와 Physical Address를 왜 분리하나요?

### Q2
Process A와 Process B가 모두 `0x1000`을 사용해도 충돌하지 않을 수 있는 이유는 무엇인가요?

### Q3
Page Table과 MMU의 역할 차이를 설명해보세요.

### Q4
TLB는 왜 필요한가요? TLB miss가 발생하면 어떤 일이 일어날 수 있나요?

## B. 심화

### Q5
Virtual Memory를 “RAM 부족 시 디스크를 쓰는 기술”이라고만 설명하면 왜 부족한가요?

### Q6
Copy-on-Write가 무엇이며 왜 효율적인가요?

### Q7
Memory-mapped File은 일반 File I/O와 어떤 관점에서 다른가요?

### Q8
Context Switch가 TLB 효율에 영향을 줄 수 있는 이유를 설명해보세요.

## C. 백엔드

### Q9
CLR Managed Heap도 결국 OS Virtual Memory 위에서 동작한다는 말은 무슨 뜻인가요?

### Q10
Process의 Virtual Size와 실제 Physical Memory 사용량이 다를 수 있는 이유는 무엇인가요?

### Q11
Container Memory Limit과 Virtual Memory를 함께 볼 때 주의할 점은 무엇인가요?

## D. 기술면접

### Q12
“Virtual Memory란 무엇인가요?”에 60초 이내로 답해보세요.

### Q13
“Page Table이 있는데 TLB가 왜 또 필요한가요?”라는 꼬리 질문에 답해보세요.

## 평가 기준

- Virtual/Physical Address를 정확히 구분하는가
- Page Table / MMU / TLB 역할을 구분하는가
- Virtual Memory를 Swap과 동일시하지 않는가
- Isolation, Protection, Demand Paging을 설명할 수 있는가
- 백엔드 Runtime/Container와 연결할 수 있는가
