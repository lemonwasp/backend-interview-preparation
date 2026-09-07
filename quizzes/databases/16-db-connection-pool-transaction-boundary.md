# 16. DB Connection Pool / Transaction Boundary — Quiz

## Core

1. DB Connection Pool이 필요한 이유를 설명하세요.
2. Connection Pool이 성능 최적화이면서 동시에 보호 장치인 이유는 무엇인가요?
3. Pool Exhaustion의 대표 원인 5가지를 말하세요.
4. App Instance 20개, 각 Pool Size 50이면 왜 단순히 `50 connections` 문제라고 보면 안 되나요?
5. Pool Size를 크게 만들면 왜 오히려 DB가 더 느려질 수 있나요?

## Transaction Boundary

6. Transaction Boundary란 무엇인가요?
7. 외부 HTTP API 호출을 긴 DB Transaction 안에 넣는 것이 위험한 이유를 설명하세요.
8. HTTP Request 하나와 DB Transaction 하나가 항상 1:1인가요?
9. Transaction이 길어지면 Lock, MVCC, Connection Pool에 각각 어떤 영향을 주나요?
10. 어떤 작업들을 같은 Transaction으로 묶을지 판단하는 기준은 무엇인가요?

## Scenario

11. Query latency가 Lock 때문에 20ms → 5초로 증가한 뒤 API 전체가 느려졌습니다. Pool Exhaustion까지 이어지는 인과관계를 설명하세요.
12. Connection acquisition latency는 높지만 실제 실행 Query latency는 낮습니다. 어떤 원인들을 의심하겠습니까?
13. DB가 CPU 100%인데 Pool을 2배로 늘리자는 제안이 나왔습니다. 어떻게 평가하겠습니까?
14. ORM의 Context/Session 객체와 물리 DB Connection을 완전히 같은 개념으로 보면 왜 위험한가요?
15. 60초 안에 Connection Pool과 Transaction Boundary의 관계를 설명하세요.

## 평가 기준

- Pool reuse와 concurrency limit을 모두 설명한다.
- Pool Size를 전체 Instance 수와 DB capacity 관점에서 본다.
- 긴 Transaction → Lock/Connection 점유 → Pool 고갈 인과관계를 설명한다.
- 외부 I/O와 Local DB Transaction의 경계를 구분한다.

상태: **답변 대기**
