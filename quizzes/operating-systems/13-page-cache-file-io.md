# 13. Page Cache와 File I/O — 이해도 확인

상태: **답변 대기**

## A. 핵심 개념

### Q1
파일을 읽는다고 해서 항상 Physical Disk를 읽는 것은 아닌 이유를 설명해보세요.

### Q2
Page Cache의 역할은 무엇인가요?

### Q3
Cache hit와 miss에서 Read 흐름이 어떻게 달라지나요?

### Q4
Dirty Page란 무엇인가요?

## B. Write와 Durability

### Q5
`write()`가 성공했다고 해서 데이터가 Storage에 영구 저장되었다고 단정할 수 없는 이유는 무엇인가요?

### Q6
`fsync`가 왜 비용이 큰가요?

### Q7
DB가 WAL과 Group Commit 같은 전략을 쓰는 이유를 File I/O 관점에서 설명해보세요.

## C. I/O 패턴

### Q8
Buffered I/O와 Direct I/O의 차이를 설명해보세요.

### Q9
Sequential I/O와 Random I/O는 왜 성능 특성이 다른가요?

### Q10
작은 I/O를 매우 많이 호출하는 것이 비효율적일 수 있는 이유는 무엇인가요?

## D. 실무 연결

### Q11
TIFF-to-PDF 최적화를 “물리 디스크 접근을 없앴다”고 단정하면 부정확할 수 있는 이유는 무엇인가요?

### Q12
`MemoryStream` 방식으로 바꾼 개선을 Page Cache와 File System 관점에서 더 정확히 설명해보세요.

### Q13
DB 자체 Buffer Pool과 OS Page Cache가 동시에 존재하면 어떤 문제가 생길 수 있나요?

## E. 기술면접

### Q14
“Page Cache란 무엇인가요?”에 45초 이내로 답해보세요.

### Q15
“write가 성공했는데 왜 데이터가 유실될 수 있나요?”라는 꼬리 질문에 답해보세요.

## 평가 기준

- Page Cache와 Physical Storage를 구분하는가
- Buffered Write와 Durability를 구분하는가
- fsync 비용을 설명할 수 있는가
- Buffering/Batching의 목적을 이해하는가
- 실제 File I/O 최적화를 과장 없이 설명하는가
