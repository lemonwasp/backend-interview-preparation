# 12. Paging과 Page Fault — 이해도 확인

상태: **답변 대기**

## A. 핵심 개념

### Q1
Page와 Frame의 차이를 설명해보세요.

### Q2
Demand Paging은 왜 필요한가요?

### Q3
Page Fault가 항상 프로그램 오류를 의미하지 않는 이유는 무엇인가요?

### Q4
Page Fault가 발생했을 때 Kernel이 어떤 순서로 처리하는지 설명해보세요.

## B. 성능

### Q5
Minor Page Fault와 Major Page Fault의 차이를 설명해보세요.

### Q6
Major Page Fault가 일반적으로 비싼 이유는 무엇인가요?

### Q7
Working Set이란 무엇인가요?

### Q8
Thrashing은 왜 발생하며 어떤 증상으로 나타날 수 있나요?

## C. 백엔드 실무

### Q9
대용량 in-memory Cache를 크게 잡았을 때 얻는 이점과 OS Memory 관점의 비용을 설명해보세요.

### Q10
대용량 파일을 한 번에 메모리로 읽는 방식과 Streaming 방식의 차이를 Working Set 관점에서 설명해보세요.

### Q11
`MemoryStream` 최적화가 File I/O는 줄이지만 Memory Pressure를 높일 수 있는 이유는 무엇인가요?

## D. 기술면접

### Q12
“Page Fault란 무엇인가요?”에 45초 이내로 답해보세요.

### Q13
“Page Fault가 많으면 무조건 나쁜가요?”라는 꼬리 질문에 답해보세요.

## 평가 기준

- Page/Frame을 구분하는가
- Page Fault를 오류와 동일시하지 않는가
- Minor/Major Fault의 비용 차이를 설명하는가
- Working Set과 Thrashing을 연결하는가
- File I/O 최적화와 Memory Pressure의 trade-off를 설명하는가
