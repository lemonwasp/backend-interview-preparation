# 13. Forward Proxy와 Reverse Proxy

## 학습 목표

- Forward Proxy와 Reverse Proxy의 차이를 설명할 수 있다.
- 각 Proxy가 누구를 대신하는지 설명할 수 있다.
- Reverse Proxy가 TLS, Routing, Cache, Security에 어떻게 쓰이는지 설명할 수 있다.
- `X-Forwarded-For` 같은 Header의 의미와 위험을 이해한다.

---

## 1. Proxy란 무엇인가?

Proxy는 Client와 Server 사이에서 요청을 대신 전달하는 중간자입니다.

```text
Client → Proxy → Server
```

중요한 질문은 하나입니다.

> Proxy가 누구를 대신하는가?

이 기준으로 Forward Proxy와 Reverse Proxy를 나눌 수 있습니다.

---

## 2. Forward Proxy

Forward Proxy는 **Client 측을 대신합니다.**

```text
Client
  ↓
Forward Proxy
  ↓
Internet Server
```

외부 Server 입장에서는 실제 Client 대신 Proxy가 연결한 것처럼 보일 수 있습니다.

사용 예:

- 사내 인터넷 접근 제어
- URL Filtering
- Client IP 숨김
- Egress 제어
- 일부 Cache

즉 Forward Proxy는 보통 Client가 자신의 Proxy를 알고 있거나 설정합니다.

---

## 3. Reverse Proxy

Reverse Proxy는 **Server 측을 대신합니다.**

```text
Client
  ↓
Reverse Proxy
  ↓
Backend A
Backend B
Backend C
```

Client는 뒤에 여러 Backend가 있다는 사실을 몰라도 됩니다.

Reverse Proxy가 할 수 있는 일:

- TLS Termination
- Host / Path 기반 Routing
- Load Balancing
- Compression
- Cache
- Authentication 연계
- Rate Limiting
- WAF 연계
- Backend IP 은닉

Nginx, Envoy 같은 소프트웨어가 대표적인 예입니다.

---

## 4. Reverse Proxy와 Application Server

예를 들어:

```text
Client
  ↓ HTTPS
Nginx
  ↓ HTTP
ASP.NET Core App
```

Nginx가 TLS를 처리하고 `/api` 요청만 Backend로 보낼 수 있습니다.

또 `/static`은 직접 제공하거나 Cache할 수도 있습니다.

이렇게 각 Layer의 책임을 나누면 Backend가 비즈니스 로직에 집중하기 쉬워집니다.

---

## 5. Client IP 문제

Reverse Proxy를 거치면 Backend Socket 관점에서는 직접 연결한 상대가 Proxy일 수 있습니다.

그래서 원래 Client IP를 전달하기 위해 다음과 같은 Header를 사용할 수 있습니다.

```text
X-Forwarded-For
Forwarded
```

하지만 아무 Client가 임의로 Header를 넣을 수도 있으므로 무조건 신뢰해서는 안 됩니다.

신뢰할 수 있는 Proxy가 어떤 Header를 덮어쓰고 추가하는지에 대한 명확한 설정이 필요합니다.

---

## 6. Proxy가 늘어나면 생기는 비용

Proxy는 기능을 제공하지만 중간 Hop이 하나 늘어납니다.

잠재적 비용:

- Network hop 증가
- Queueing
- Connection 관리 비용
- TLS Termination 비용
- 잘못된 Timeout 설정
- Header 전달 문제
- 장애 지점 추가

따라서 Proxy를 쓰는 목적과 관측성을 명확히 해야 합니다.

---

## 7. Forward vs Reverse Proxy 핵심 비교

| 구분 | Forward Proxy | Reverse Proxy |
|---|---|---|
| 대신하는 쪽 | Client | Server |
| 주 사용 목적 | Egress 제어, 익명화, 필터링 | Routing, TLS, LB, Cache |
| 누가 보통 알고 있나 | Client가 설정 | Client는 뒤 구조를 몰라도 됨 |

---

## 8. 60초 면접 답변

> Forward Proxy는 Client를 대신해 외부 Server에 요청하는 Proxy이고, Reverse Proxy는 Server를 대신해 Client 요청을 받아 내부 Backend로 전달하는 Proxy입니다. Forward Proxy는 사내 Egress 제어나 필터링에 자주 쓰이고, Reverse Proxy는 TLS Termination, Routing, Load Balancing, Cache, Rate Limiting 등에 사용됩니다. Reverse Proxy를 거치면 Backend가 보는 연결 상대는 Proxy이므로 실제 Client IP를 전달할 때 Forwarded 계열 Header를 사용할 수 있지만, 신뢰할 수 있는 Proxy가 설정한 값만 신뢰해야 합니다.

---

## 핵심 요약

```text
Forward Proxy = Client 대신
Reverse Proxy = Server 대신
```

다음 주제: Load Balancing
