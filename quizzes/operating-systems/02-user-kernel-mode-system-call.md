# 02. 사용자 모드, 커널 모드, System Call — 이해도 확인

상태: **답변 대기**

문서를 보지 않고 자신의 말로 답합니다. 정의를 외우는 것보다
**왜 권한을 나누는지 → 어떻게 Kernel 기능을 요청하는지 → 성능에 어떤 비용이
생기는지**를 연결해서 설명하는 것이 목표입니다.

## A. 파인만 확인 질문

### Q1

컴퓨터를 전혀 모르는 사람에게 User Mode와 Kernel Mode를 왜 나누는지
설명해보세요.

### Q2

User Mode 프로그램이 마음대로 할 수 없는 작업에는 어떤 것들이 있나요?
왜 제한해야 하나요?

### Q3

System Call이 무엇인지 한 문장으로 설명한 뒤, 파일 읽기를 예로 들어
전체 흐름을 설명해보세요.

### Q4

C#의 `File.ReadAllBytes()` 같은 라이브러리 API와 System Call을 완전히 같은
것이라고 보면 안 되는 이유는 무엇인가요?

## B. 구분 문제

### Q5

Mode Switch와 Context Switch의 차이를 설명해보세요.

### Q6

다음 문장이 맞는지 판단하고 이유를 설명하세요.

> System Call이 한 번 발생하면 반드시 다른 Thread로 Context Switch가 발생한다.

### Q7

파일 `read()` 요청이 Page Cache에서 바로 처리되어 같은 Thread로 돌아왔다고
가정합니다. 이 경우 Mode Switch와 Context Switch는 각각 발생했을 가능성이
어떻게 다른가요?

## C. 실무 연결 질문

### Q8

TIFF-to-PDF 처리에서 임시 PNG 파일을 없애고 `MemoryStream`으로 바꾼 개선을
User/Kernel 경계와 File System 관점에서 설명해보세요.

### Q9

“System Call 수를 줄였으니 무조건 빨라집니다”라는 주장에 어떤 문제가 있나요?
실제 성능을 판단할 때 함께 봐야 할 요소를 두 가지 이상 말해보세요.

### Q10

백엔드 HTTP 서버에서 Network I/O가 애플리케이션 코드와 Kernel 사이를 어떻게
오가는지 간단한 흐름으로 설명해보세요.

## D. 기술면접 질문

### Q11

“User Mode와 Kernel Mode의 차이가 무엇인가요?”에 60초 이내로 답해보세요.

### Q12

면접관이 다음과 같이 꼬리 질문을 했습니다.

> System Call과 Context Switch는 같은 것 아닌가요?

30초 이내로 답해보세요.

### Q13

> System Call은 왜 일반 함수 호출보다 비용이 큰가요?

에 답하되 “무조건 느리다”는 식의 과장을 피해서 설명해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 보호 모델 | 권한 분리가 시스템 안정성과 보안을 위한 것임을 설명하는가 |
| 호출 흐름 | User → System Call → Kernel → User 흐름을 설명하는가 |
| 개념 구분 | Mode Switch와 Context Switch를 구분하는가 |
| 정확성 | System Call이 항상 Context Switch를 만든다고 오해하지 않는가 |
| 실무 연결 | File I/O와 Network I/O를 OS 계층과 연결하는가 |
| 성능 관점 | System Call 횟수만으로 성능을 단정하지 않는가 |

## 평가 결과

답변 제출 후 업데이트 예정입니다.

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
