# Quiz — Synchronization Primitives

상태: **답변 대기**

1. Race Condition과 Critical Section을 설명하세요.
2. `lock`은 어떤 상황에서 사용하는가요?
3. `lock` 안에서 오래 걸리는 I/O가 위험한 이유는 무엇인가요?
4. `Mutex`와 `Semaphore`의 차이를 설명하세요.
5. `SemaphoreSlim`이 backend에서 유용한 사례를 하나 말하세요.
6. `Interlocked`는 어떤 종류의 문제에 적합한가요?
7. `volatile int counter; counter++;`가 안전하지 않은 이유는 무엇인가요?
8. `lock` 블록 안에서 `await`를 사용할 수 없는 이유와 대안은 무엇인가요?
9. Global Lock과 Fine-grained Lock의 trade-off는 무엇인가요?
10. process 내부 `lock`이 여러 App Instance 간 duplicate order를 막지 못하는 이유는 무엇인가요?
11. 여러 Instance 환경에서 사용할 수 있는 동시성 제어 방법을 3개 말하세요.
12. Lock ordering이 deadlock 예방에 도움이 되는 이유는 무엇인가요?
13. `SemaphoreSlim(1,1)`을 async mutual exclusion에 사용할 때 고려할 점은 무엇인가요?

## 평가 기준

- lock/semaphore/mutex/interlocked의 역할을 구분한다.
- atomicity와 visibility를 구분한다.
- process-local 동기화와 distributed concurrency control을 구분한다.
- contention/deadlock trade-off를 설명한다.
- async 코드와 동기화 도구를 연결한다.

## 평가 결과

- 점수: Pending
- 보완점: Pending
- Re-test: Pending
