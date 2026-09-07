# 10. Memory Visibility와 Memory Barrier — 이해도 확인

상태: **답변 대기**

## A. 핵심 개념

### Q1
Memory Visibility와 Atomicity의 차이를 설명해보세요.

### Q2
멀티코어 환경에서 한 Thread가 쓴 값이 다른 Thread에 즉시 보인다고 단정할 수 없는 이유는 무엇인가요?

### Q3
Compiler/CPU Reordering이 동시성에서 왜 문제가 될 수 있나요?

### Q4
Memory Barrier가 해결하려는 문제를 쉽게 설명해보세요.

## B. C# 연결

### Q5
`volatile`은 무엇을 보장하려고 사용하며, 무엇을 보장하지 못하나요?

### Q6
`volatile int counter; counter++;`가 Thread-safe하지 않은 이유는 무엇인가요?

### Q7
`lock`, `Interlocked`, `volatile`을 각각 언제 고려할지 설명해보세요.

### Q8
`lock`이 Mutual Exclusion 외에 메모리 가시성 측면에서도 중요한 이유는 무엇인가요?

## C. 백엔드 실무

### Q9
Background Worker와 HTTP Request Thread가 같은 in-memory 상태를 공유할 때 어떤 문제가 생길 수 있나요?

### Q10
Singleton Cache 객체의 필드를 여러 Thread가 갱신한다면 무엇을 확인해야 하나요?

## D. 기술면접

### Q11
“Memory Visibility란 무엇인가요?”에 45초 이내로 답해보세요.

### Q12
“volatile을 붙이면 Race Condition이 해결되나요?”라는 꼬리 질문에 답해보세요.

## 평가 기준

- Atomicity / Visibility / Ordering을 구분하는가
- CPU Cache와 Reordering의 존재 이유를 설명하는가
- `volatile`을 만능 Thread-safe 도구로 오해하지 않는가
- `lock`과 `Interlocked`의 역할 차이를 설명하는가
- 실무 Shared State와 연결할 수 있는가
