# Quiz 27. Backpressure Deep Dive

Status: 답변 대기

## 핵심 확인

1. Backpressure를 한 문장으로 설명하세요.
2. Producer가 1000 req/s, Consumer가 200 req/s를 처리할 때 무슨 일이 생기나요?
3. 무한 Queue가 왜 위험한가요?
4. Bounded Queue는 어떤 문제를 막나요?
5. Queue가 가득 찼을 때 선택할 수 있는 전략은 무엇인가요?
6. Concurrency Limit이 DB나 Thread Pool을 보호하는 이유는 무엇인가요?
7. Rate Limiting과 Backpressure의 차이는 무엇인가요?
8. Buffering은 언제 도움이 되고 언제 근본 해결책이 아닌가요?
9. Load Shedding이 필요한 이유는 무엇인가요?
10. TCP Flow Control과 Application-level Backpressure는 어떤 점에서 비슷하고 어떤 점에서 다른가요?
11. Message Queue를 도입해도 Backpressure 문제가 사라지지 않는 이유는 무엇인가요?
12. Consumer Lag이 계속 증가한다는 것은 무엇을 의미하나요?

## 면접 꼬리 질문

13. Queue depth는 안정적인데 p99 latency가 증가한다면 무엇을 의심하겠습니까?
14. 모든 요청을 받는 것과 일부를 503으로 거절하는 것 중 후자가 더 나을 수 있는 이유는 무엇인가요?
15. DB Connection Pool 50개인 서비스에 동시에 500개 Query가 들어오면 어떤 보호 장치를 두겠습니까?
16. Retry가 Backpressure 문제를 악화시키는 과정을 설명하세요.
17. Backpressure 운영을 위해 어떤 Metrics를 보겠습니까?

## 60초 답변

18. "Backpressure가 무엇이고 왜 필요한가요?"에 60초 이내로 답하세요.
