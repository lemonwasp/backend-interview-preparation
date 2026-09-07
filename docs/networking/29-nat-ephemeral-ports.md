# 29. NAT and Ephemeral Ports

## 한 줄 요약

NAT는 내부 주소를 외부 주소로 변환하고, Ephemeral Port는 Client가 outbound connection을 만들 때 임시로 사용하는 Source Port다. 대규모 서비스에서는 이 임시 Port와 NAT mapping도 유한 자원이다.

## 출발점: Connection은 무엇으로 구분될까

TCP Connection은 보통 다음 4-tuple로 식별된다.

```text
Source IP
Source Port
Destination IP
Destination Port
```

예:

```text
10.0.0.12:53124 -> 203.0.113.10:443
```

Client가 새 연결을 만들 때 OS는 사용 가능한 임시 Source Port를 하나 선택한다.
이것이 Ephemeral Port다.

## 왜 Port가 필요한가

한 Client가 동시에 여러 Connection을 만들 수 있어야 한다.

```text
10.0.0.12:53124 -> server:443
10.0.0.12:53125 -> server:443
10.0.0.12:53126 -> server:443
```

같은 Source IP라도 Source Port가 달라서 Connection을 구분할 수 있다.

## NAT란

Private IP는 인터넷에서 그대로 Route되지 않는다.

NAT 장비는 내부 주소를 외부 주소로 변환한다.

```text
Before NAT
10.0.0.12:53124 -> 203.0.113.10:443

After NAT
198.51.100.5:61001 -> 203.0.113.10:443
```

NAT는 이 mapping을 기억해야 응답을 올바른 내부 Host로 돌려보낼 수 있다.

## PAT / NAPT

실무에서 흔히 보는 형태는 하나의 Public IP를 여러 내부 Connection이 공유하도록 Port까지 변환하는 방식이다.

```text
10.0.0.12:53124 -> public:61001
10.0.0.13:53124 -> public:61002
```

그래서 Public IP 하나가 처리할 수 있는 동시 outbound mapping 수에도 한계가 생긴다.

## Ephemeral Port Exhaustion

Client가 너무 많은 outbound Connection을 동시에 만들거나 너무 자주 새로 만들면 사용 가능한 Source Port가 부족해질 수 있다.

증상은 환경마다 다르지만 개념적으로:

```text
connect()
-> 사용할 source port 부족
-> 새 connection 실패
```

## 왜 Connection Pooling과 연결되는가

매 요청마다 Connection을 새로 만들면:

- TCP handshake 증가
- TLS handshake 증가
- TIME_WAIT 증가
- Ephemeral Port 소비 증가
- NAT mapping 증가

Connection을 재사용하면 이 모든 압력을 줄일 수 있다.

## TIME_WAIT과의 관계

Connection을 닫은 직후 Port를 무조건 즉시 재사용할 수 있는 것은 아니다.

Active closer 쪽에 TIME_WAIT이 남을 수 있다.

따라서 높은 connection churn은 많은 TIME_WAIT socket을 만들고, 특정 환경에서는 usable port pressure를 높일 수 있다.

중요한 점:

> TIME_WAIT 자체를 무조건 없애는 것이 해결책은 아니다. 먼저 불필요한 connection churn을 줄이는 것이 우선이다.

## NAT Gateway에서도 고갈이 생길 수 있다

Cloud 환경에서는 여러 App Instance가 하나의 NAT Gateway/Public IP를 공유할 수 있다.

각 App만 보면 Connection 수가 많지 않아 보여도 전체 합계는 클 수 있다.

```text
100 instances
x each 1000 outbound connections
= 100,000 connections
```

그래서 장애 원인을 App Process 내부만 보면 놓칠 수 있다.

## DNS와는 다른 문제

외부 API 연결 실패가 발생한다고 해서 모두 DNS 문제는 아니다.

가능한 계층:

- DNS resolution
- TCP connect
- NAT mapping
- Ephemeral Port
- TLS handshake
- Remote server limit

단계별로 나눠서 봐야 한다.

## Outbound Connection 폭증 시나리오

```text
Downstream latency 증가
-> 기존 요청이 오래 Connection 점유
-> 새 요청은 새 Connection 생성
-> Retry 증가
-> outbound connection 증가
-> NAT / ephemeral port pressure 증가
-> connect failure 증가
```

앞서 배운 Timeout, Retry, Pool, Circuit Breaker와 전부 연결된다.

## Connection Reuse가 중요한 이유

HTTP Client를 잘못 사용해서 매 요청마다 새 Client/Connection을 만들면 높은 트래픽에서 문제가 커질 수 있다.

C#에서는 HTTP connection pooling을 활용하는 방식으로 `HttpClient` lifecycle을 설계해야 하며, 단순히 매 요청마다 새 Connection을 만드는 습관은 피해야 한다.

## Server의 Listening Port와 Client Ephemeral Port

헷갈리기 쉬운 부분이다.

Server:

```text
:443 listen
```

Client:

```text
:53124 -> server:443
```

Server는 well-known/service port에서 대기하고, Client는 outbound connection마다 ephemeral source port를 사용한다.

## NAT가 TCP reliability를 제공하는가

아니다.

NAT는 주소/Port 변환과 mapping을 담당한다.
TCP의 reliability, ordering, retransmission은 TCP의 역할이다.

## 실무에서 볼 것

- Outbound connection count
- Connection reuse ratio
- TIME_WAIT count
- Connect error rate
- NAT gateway connection metrics
- Remote destination별 connection 수
- Retry rate

Linux 환경에서는 `ss` 같은 도구로 socket state를 관찰할 수 있다.

## 흔한 오해

### "Port는 65535개니까 무조건 65535 Connection까지만 가능하다"

실제 한계는 OS 설정, 사용 가능한 ephemeral range, destination tuple, NAT 구조 등에 따라 달라진다. 단순 숫자 하나로 외우지 않는다.

### "TIME_WAIT이 많으면 무조건 OS tuning부터 한다"

먼저 왜 Connection을 그렇게 자주 만들고 닫는지 확인해야 한다.

### "NAT 문제면 Application과 무관하다"

Application의 connection reuse, retry 정책이 NAT pressure를 직접 키울 수 있다.

## 60초 면접 답변

> Client가 TCP outbound connection을 만들 때 OS는 임시 Source Port인 ephemeral port를 사용합니다. NAT 환경에서는 private IP와 source port가 public IP와 다른 port로 매핑되고 NAT 장비가 이 상태를 유지합니다. 높은 트래픽에서 매 요청마다 새 connection을 만들면 handshake뿐 아니라 TIME_WAIT, ephemeral port, NAT mapping이 증가해 새 connection 생성 실패로 이어질 수 있습니다. 그래서 connection pooling과 keep-alive가 중요하고, 장애 분석 시 application connection 수뿐 아니라 TIME_WAIT, connect error, NAT gateway metrics와 retry rate를 함께 봐야 합니다.
