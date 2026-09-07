# 09. Race Condition, Lock, Deadlock — 이해도 확인

상태: **답변 대기**

## A. Race Condition

### Q1
`counter++`가 Race Condition을 일으킬 수 있는 이유를 read-modify-write 단계로 설명해보세요.

### Q2
Race Condition이 발생하기 쉬운 세 조건을 말해보세요.

### Q3
Critical Section이 무엇인지 설명하고 백엔드 예시를 하나 들어보세요.

### Q4
Shared Mutable State를 줄이는 것이 왜 강력한 동시성 전략인가요?

## B. Lock / Atomic

### Q5
Lock/Mutex가 어떤 문제를 해결하나요?

### Q6
Lock을 너무 넓게 잡으면 어떤 성능 문제가 생길 수 있나요?

### Q7
Critical Section을 너무 잘게 나누면 어떤 새로운 위험이 생길 수 있나요?

### Q8
`Interlocked.Increment` 같은 Atomic Operation과 일반 Lock의 차이는 무엇인가요?

### Q9
단순 Atomic Operation 하나로 복잡한 비즈니스 Transaction 전체를 보호할 수 없는 이유는 무엇인가요?

### Q10
Mutex와 Semaphore를 동시 접근 허용 개수 관점에서 설명해보세요.

## C. Deadlock

### Q11
Thread A가 Lock 1을, Thread B가 Lock 2를 가진 상황에서 Deadlock이 어떻게 발생할 수 있는지 설명해보세요.

### Q12
Deadlock의 4가지 필요 조건을 모두 말하고 각각 설명해보세요.

### Q13
Global Lock Ordering이 왜 Circular Wait를 막을 수 있나요?

### Q14
Timeout이나 Try-Lock을 사용하면 어떤 장점과 새로운 설계 문제가 생기나요?

### Q15
Database Transaction에서도 Deadlock이 발생할 수 있는 예를 들어보세요.

## D. 백엔드 실무

### Q16
재고가 1개일 때 두 요청이 동시에 구매에 성공하는 문제를 설명해보세요.

### Q17
여러 서버 Instance가 같은 DB를 사용할 때 C# `lock`만으로 재고 Race Condition을 막을 수 없는 이유는 무엇인가요?

### Q18
Cache Stampede가 무엇이고 Lock 또는 Single-flight가 어떻게 완화할 수 있나요?

### Q19
TIFF-to-PDF 페이지 변환을 병렬화할 때 최종 PDF 객체가 Thread-safe하지 않다면 어떤 구조로 바꿀 수 있나요?

## E. 기술면접 질문

### Q20
“Race Condition이 무엇인가요?”에 30초 이내로 답해보세요.

### Q21
“Lock을 쓰면 무조건 Thread-safe한가요?”라는 꼬리 질문에 답해보세요.

### Q22
“Deadlock의 4가지 조건은 무엇인가요?”에 자료 없이 답해보세요.

### Q23
“Deadlock을 실무에서 어떻게 예방하나요?”에 60초 이내로 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| Race 이해 | 실행 순서와 shared mutable state를 연결하는가 |
| Lock 이해 | mutual exclusion과 contention을 함께 설명하는가 |
| Atomic 이해 | 단순 원자 연산과 복합 transaction을 구분하는가 |
| Deadlock | 4가지 조건과 lock ordering을 설명하는가 |
| 범위 인식 | in-process lock과 DB/distributed concurrency를 구분하는가 |
| 실무 연결 | 재고, cache, PDF 병렬화 사례로 설명할 수 있는가 |

## 평가 결과

답변 제출 후 업데이트 예정입니다.

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
