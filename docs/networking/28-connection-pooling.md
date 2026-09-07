# 28. Connection Pooling

## 한 줄 요약

Connection Pooling은 DB나 HTTP 서버와의 연결을 매 요청마다 새로 만들지 않고 **미리 만든 연결을 재사용**해 handshake 비용과 자원 고갈을 줄이는 기술이다.

## 왜 필요한가

새 Connection은 공짜가 아니다.

TCP 연결이라면:

```text
TCP 3-way handshake
+ TLS handshake(HTTPS라면)
+ Kernel socket / buffer
+ FD
+ NAT mapping 가능성
```

DB라면 추가로:

- 인증
- Session 초기화
- DB backend/process/thread 자원

등이 필요할 수 있다.

매 요청마다 연결을 만들고 버리면 latency와 CPU 비용이 증가한다.

## Pool의 기본 구조

```text
Application
   |
   +-- Connection 1 [idle]
   +-- Connection 2 [in use]
   +-- Connection 3 [in use]
   +-- Connection 4 [idle]
```

요청이 오면:

1. Pool에서 idle Connection을 빌린다.
2. 작업을 수행한다.
3. Connection을 닫는 대신 Pool에 반환한다.

## Pool이 없을 때

```text
Request
 -> connect
 -> handshake
 -> request/query
 -> close
```

Pool이 있으면:

```text
Request
 -> borrow existing connection
 -> request/query
 -> return
```

## Pool Size가 너무 작으면

동시에 100개 요청이 들어왔는데 Pool이 10개라면 90개는 기다릴 수 있다.

결과:

- Queue wait 증가
- p95/p99 latency 증가
- Timeout

## Pool Size가 너무 크면

"많을수록 좋다"도 틀리다.

DB Connection Pool을 1000개로 만들면:

- DB가 1000개 Query를 동시에 처리해야 할 수 있음
- Context switching 증가
- Memory 증가
- Lock contention 증가
- DB 자체 max_connections 초과

Pool은 downstream capacity에 맞춰야 한다.

## Connection Pool은 Backpressure 장치이기도 하다

Pool 크기가 제한되어 있으면 Application이 downstream에 무제한 동시 요청을 보내지 못한다.

따라서 Pool은 단순 성능 최적화뿐 아니라 **Concurrency Boundary** 역할도 한다.

## Pool Acquisition Timeout

Connection을 빌릴 수 없을 때 무한정 기다리면 안 된다.

```text
Pool exhausted
-> wait 30s
-> upstream timeout 2s
```

이런 구성은 이미 의미가 없다.

Connection acquisition timeout도 전체 timeout budget 안에 들어가야 한다.

## Idle / Lifetime 관리

장기 Connection은 중간 장비나 서버 정책 때문에 끊길 수 있다.

그래서 Pool에는 보통:

- idle timeout
- max lifetime
- health validation

같은 정책이 있다.

너무 오래된 Connection을 계속 재사용하면 stale connection error가 날 수 있다.

## HTTP Connection Pooling

HTTP Client도 매 요청마다 새 객체/Connection을 만드는 방식은 좋지 않을 수 있다.

HTTP/1.1에서는 Keep-Alive로 TCP Connection을 재사용한다.
HTTP/2에서는 하나의 Connection에서 여러 Stream을 multiplex할 수 있다.

즉 HTTP Version에 따라 "적정 Connection 수"의 의미도 달라진다.

## DB Connection Pooling

DB Connection은 보통 HTTP Connection보다 더 비싼 자원이다.

고려할 것:

- Application instance 수
- instance당 pool size
- DB max connections

예:

```text
App 20대
각 Pool 50
= 최대 1000 DB connections
```

Instance 하나만 보고 Pool Size를 정하면 전체 DB capacity를 초과할 수 있다.

## Pool Exhaustion

증상:

- Connection acquisition time 증가
- Timeout 증가
- Pool active = max
- idle = 0

원인 후보:

- 느린 Query
- Connection leak
- Transaction이 너무 오래 유지됨
- downstream 장애
- 트래픽 급증

## Connection Leak

빌린 Connection을 반환하지 않으면 Pool이 점점 고갈된다.

C#에서는 DB Connection 등을 `using`/`await using`으로 수명 관리하는 습관이 중요하다.

## Little's Law 감각

아주 단순화하면 동시성은 처리율과 응답시간에 비례한다.

```text
Concurrency ≈ Throughput × Latency
```

Downstream latency가 늘면 같은 throughput에서도 필요한 in-flight connection 수가 늘 수 있다.

하지만 Pool을 무작정 키우기 전에 latency 증가 원인을 먼저 봐야 한다.

## Connection Pool vs Thread Pool

둘 다 재사용과 자원 상한을 제공하지만 대상이 다르다.

- Thread Pool: 실행 Thread
- Connection Pool: 외부 resource connection

둘 중 하나가 먼저 고갈될 수 있고 서로 영향을 준다.

## Backend 장애 시나리오

```text
DB 느려짐
-> Query latency 증가
-> Connection이 오래 점유됨
-> Pool exhausted
-> Request wait 증가
-> Timeout
-> Retry
-> DB 부하 더 증가
```

따라서 Pool만 보는 것이 아니라 Timeout, Retry, Circuit Breaker, Bulkhead를 함께 봐야 한다.

## 60초 면접 답변

> Connection Pooling은 DB나 HTTP 서버와의 연결을 매 요청마다 새로 생성하지 않고 재사용하는 방식입니다. TCP/TLS handshake와 인증 비용을 줄여 latency를 낮추고, 동시에 사용할 Connection 수에 상한을 두어 downstream을 보호하는 역할도 합니다. Pool이 너무 작으면 acquisition wait와 timeout이 증가하고, 너무 크면 DB나 downstream의 동시 처리량을 초과해 오히려 contention과 장애를 키울 수 있습니다. 실무에서는 전체 application instance 수를 포함해 pool size를 계산하고, acquisition timeout, idle timeout, max lifetime, connection leak, active/idle/wait metrics를 함께 모니터링해야 합니다.
