# 19. TCP가 OS에서 처리되는 과정 — 이해도 확인

상태: **답변 대기**

## A. 파인만 확인 질문

### Q1
TCP의 핵심 처리가 일반적으로 어디에서 수행되는지 설명해보세요.

### Q2
Packet이 NIC에 도착한 뒤 애플리케이션이 읽기까지의 흐름을 순서대로 설명해보세요.

### Q3
애플리케이션이 `send()`를 호출한 뒤 실제 네트워크로 나가기까지의 흐름을 설명해보세요.

### Q4
TCP가 애플리케이션에 제공하는 것은 packet인가요, ordered byte stream인가요?

## B. TCP 상태 질문

### Q5
Packet이 순서가 바뀌어 도착했을 때 TCP는 어떻게 처리하나요?

### Q6
Packet loss가 발생했을 때 retransmission은 누가 담당하나요?

### Q7
`send()` 성공이 상대방 애플리케이션의 수신 완료를 의미하지 않는 이유는 무엇인가요?

### Q8
Socket Receive Buffer가 가득 차면 TCP Flow Control과 어떻게 연결될 수 있나요?

### Q9
Flow Control과 Congestion Control의 목적 차이를 설명해보세요.

### Q10
TIME_WAIT가 단순한 낭비 상태가 아닌 이유는 무엇인가요?

## C. 백엔드 연결 질문

### Q11
accept queue saturation과 socket receive buffer saturation은 어떻게 다른 문제인가요?

### Q12
Retransmission 증가를 발견했다면 애플리케이션 코드만 봐서는 안 되는 이유는 무엇인가요?

### Q13
대량의 짧은 HTTP connection에서 TIME_WAIT가 많이 보일 수 있는 이유는 무엇인가요?

### Q14
지금까지 배운 FD, DMA, Interrupt, I/O Multiplexing이 TCP 처리 흐름과 어떻게 연결되는지 설명해보세요.

## D. 기술면접 질문

### Q15
“TCP packet이 서버 애플리케이션까지 도달하는 과정을 설명해주세요.”에 90초 이내로 답해보세요.

### Q16
“TCP는 reliable하니까 packet loss가 없다는 뜻인가요?”라는 꼬리 질문에 답해보세요.

### Q17
“TCP는 message boundary를 보장하나요?”에 답해보세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| 전체 흐름 | NIC → Kernel → TCP → Socket → App을 설명하는가 |
| Reliability | loss 자체가 없다는 의미로 오해하지 않는가 |
| Stream | TCP가 ordered byte stream임을 이해하는가 |
| Buffer | Send/Receive Buffer의 의미를 설명하는가 |
| Control | Flow Control과 Congestion Control을 구분하는가 |
| OS 연결 | FD/DMA/Interrupt/I/O multiplexing을 연결하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
