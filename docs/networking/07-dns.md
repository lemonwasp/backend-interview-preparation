# 07. DNS

## 이번 학습 목표

- 도메인 이름이 IP 주소로 해석되는 흐름을 설명한다.
- Recursive Resolver와 Authoritative DNS의 역할을 구분한다.
- DNS Cache와 TTL이 성능·장애 대응에 미치는 영향을 설명한다.

---

## 1. DNS는 왜 필요한가?

사람은 `api.example.com` 같은 이름을 기억하기 쉽지만 네트워크 통신은 결국 IP 주소를 사용합니다.

DNS는 이름을 네트워크 주소 등으로 매핑하는 분산 계층형 시스템입니다.

```text
api.example.com
      ↓ DNS
203.0.113.10
```

---

## 2. 전체 질의 흐름

일반적인 흐름을 단순화하면 다음과 같습니다.

```text
Application
  ↓
OS / local cache
  ↓
Recursive Resolver
  ↓
Root DNS
  ↓
TLD DNS (.com)
  ↓
Authoritative DNS
  ↓
IP address
```

실제로 Root부터 매번 조회하는 것은 아닙니다. 여러 계층의 Cache가 존재합니다.

---

## 3. Recursive Resolver와 Authoritative Server

### Recursive Resolver

클라이언트를 대신해 필요한 DNS 정보를 찾아주는 서버입니다.

### Authoritative DNS

특정 도메인에 대한 최종 권한 있는 DNS 정보를 제공합니다.

예를 들어 `example.com`의 A/AAAA/CNAME 레코드에 대한 최종 답을 관리합니다.

---

## 4. 주요 Record

| Record | 역할 |
|---|---|
| A | IPv4 주소 |
| AAAA | IPv6 주소 |
| CNAME | 다른 이름으로 alias |
| MX | 메일 서버 |
| NS | authoritative name server |
| TXT | 문자열 정보, 인증 등에 사용 |

백엔드 서비스에서는 A/AAAA/CNAME을 특히 자주 접합니다.

---

## 5. TTL과 Cache

DNS 응답에는 TTL(Time To Live)이 있어 Cache가 얼마나 오래 결과를 보관할지 결정합니다.

긴 TTL:
- Resolver 부하 감소
- DNS latency 감소
- 변경 반영 느림

짧은 TTL:
- 변경 반영 빠름
- Query 증가 가능

따라서 장애 전환이나 IP 변경을 계획할 때 TTL은 중요한 운영 변수입니다.

---

## 6. DNS 장애가 왜 애플리케이션 장애처럼 보일까?

애플리케이션이 정상이어도 DNS 해석이 실패하면 클라이언트는 서버의 IP를 얻지 못합니다.

```text
Client
  ↓ DNS failure
no destination IP
  ↓
connection cannot start
```

그래서 HTTP timeout처럼 보이는 문제의 원인이 실제로는 DNS일 수 있습니다.

---

## 7. UDP와 TCP

전통적인 DNS 질의는 UDP/53을 많이 사용합니다. 하지만 응답 크기나 특정 상황에서는 TCP가 사용될 수 있으며, DNS over TLS/HTTPS 같은 방식도 존재합니다.

따라서 “DNS는 항상 UDP”라고 단정하면 안 됩니다.

---

## 8. Backend와 DNS

다음 상황에서 DNS 이해가 중요합니다.

- Service discovery
- Load balancing
- Failover
- CDN
- Kubernetes service resolution
- DB endpoint 변경
- API endpoint migration

DNS는 단순 전화번호부가 아니라 분산 시스템의 트래픽 라우팅과 장애 대응에도 영향을 줍니다.

---

## 9. 60초 면접 답변

> DNS는 사람이 사용하는 도메인 이름을 IP 주소 같은 네트워크 정보로 해석하는 분산 계층형 시스템입니다. 클라이언트는 보통 로컬 캐시와 Recursive Resolver를 거치고, 필요한 경우 Root·TLD·Authoritative DNS 계층을 따라가 최종 레코드를 얻습니다. 결과는 TTL 동안 캐시될 수 있어서 latency와 DNS 부하를 줄이지만, 변경 반영이 늦어지는 trade-off가 있습니다. DNS 장애가 발생하면 애플리케이션이 정상이어도 연결할 IP를 얻지 못해 서비스 장애처럼 보일 수 있습니다.

---

## 핵심 요약

```text
DNS = distributed hierarchical name resolution
Resolver ≠ Authoritative server
TTL = cache duration + operational trade-off
```

다음 주제: HTTP/1.1
