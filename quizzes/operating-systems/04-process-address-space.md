# 04. Process Address Space — 이해도 확인

상태: **답변 대기**

문서를 보지 않고 자신의 말로 답합니다.

## A. 파인만 확인 질문

### Q1

Process Address Space가 무엇인지 설명해보세요.

### Q2

Code, Data, Heap, Stack 영역을 각각 한 문장으로 설명해보세요.

### Q3

Stack과 Heap은 무엇이 다른가요?

### Q4

왜 Thread마다 Stack은 따로 두고 Heap은 공유할 수 있나요?

## B. Virtual Memory

### Q5

Process마다 독립적인 Virtual Address Space가 필요한 이유는 무엇인가요?

### Q6

Process A와 Process B가 모두 `0x1000` 주소를 사용할 수 있는 이유를 설명해보세요.

### Q7

Virtual Address가 Physical Address로 변환되는 흐름을 간단히 설명해보세요.

### Q8

Page Fault는 왜 항상 프로그램 오류를 의미하지 않나요?

## C. 백엔드 연결

### Q9

OOM, Memory Leak, Stack Overflow의 차이를 설명해보세요.

### Q10

C#의 Managed Heap과 OS Virtual Memory를 같은 것으로 보면 안 되는 이유는 무엇인가요?

### Q11

TIFF-to-PDF에서 임시 파일을 `MemoryStream`으로 바꾸면 어떤 성능상 이점과 메모리상 Trade-off가 생길 수 있나요?

## D. 기술면접 질문

### Q12

“Process의 메모리 구조를 설명해주세요”에 60초 이내로 답해보세요.

### Q13

“Virtual Memory는 RAM이 부족할 때 디스크를 쓰는 기술 아닌가요?”라는 꼬리 질문에 답해보세요.

### Q14

“지역 변수는 전부 Stack에 있나요?”라는 질문에 과장 없이 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 구조 이해 | Code/Data/Heap/Stack 역할을 구분하는가 |
| 가상 메모리 | Virtual/Physical Address를 구분하는가 |
| 격리 이해 | Process별 주소 공간이 왜 필요한지 설명하는가 |
| Page Fault | 정상적 Demand Paging과 오류를 구분하는가 |
| Runtime 구분 | CLR Managed Heap과 OS 메모리를 동일시하지 않는가 |
| 실무 연결 | OOM, Stack Overflow, MemoryStream Trade-off에 연결하는가 |

## 평가 결과

답변 제출 후 업데이트 예정입니다.

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
