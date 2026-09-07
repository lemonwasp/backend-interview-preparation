# 09. HTTP/2 — 이해도 확인

상태: **답변 대기**

## A. 기본 개념

### Q1
HTTP/2가 HTTP/1.1에 비해 개선한 핵심 문제는 무엇인가요?

### Q2
Binary Framing이 무엇인지 설명하세요.

### Q3
HTTP/2 Stream과 TCP Connection의 관계를 설명하세요.

### Q4
Multiplexing이 HTTP/1.1의 HOL Blocking을 어떻게 줄이나요?

## B. 남아 있는 한계

### Q5
HTTP/2에서도 TCP-level HOL Blocking이 남는 이유는 무엇인가요?

### Q6
한 TCP Segment가 유실되면 서로 다른 HTTP/2 Stream도 영향을 받을 수 있는 이유를 설명하세요.

### Q7
HTTP/2가 하나의 Connection을 오래 재사용할 때 장점과 잠재적 단점을 설명하세요.

### Q8
HPACK이 무엇을 해결하려는 기술인가요?

## C. 실무와 면접

### Q9
gRPC가 HTTP/2와 잘 맞는 이유를 설명하세요.

### Q10
HTTP/1.1 → HTTP/2 → HTTP/3의 개선 방향을 HOL Blocking 관점에서 60초 이내로 설명하세요.

## 평가 기준

| 항목 | 확인 내용 |
|---|---|
| Multiplexing | logical stream과 TCP connection을 구분하는가 |
| HOL | application-level과 TCP-level HOL을 구분하는가 |
| Framing | frame 기반 interleaving을 설명하는가 |
| 실무 | gRPC/proxy와 연결하는가 |

## 평가 결과

- 이해도:
- 강점:
- 부족한 부분:
- 추가 학습:
- 재시험 날짜:
