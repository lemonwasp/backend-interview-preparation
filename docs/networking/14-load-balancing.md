# 14. Load Balancing

## 학습 목표

- Load Balancer가 왜 필요한지 설명할 수 있다.
- L4와 L7 Load Balancing의 차이를 설명할 수 있다.
- Round Robin, Least Connections, Hash 기반 분산을 비교할 수 있다.
- Health Check와 Failover의 관계를 설명할 수 있다.
- Sticky Session의 장단점을 설명할 수 있다.

---

## 1. Load Balancer는 왜 필요한가?

서버가 한 대뿐이면 모든 요청이 한곳에 집중됩니다.

```text
Client → Server A
```

트래픽이 늘면 처리량 한계와 Single Point of Failure 문제가 생깁니다.

Load Balancer를 두면 여러 Backend에 요청을 분산할 수 있습니다.

```text
Clients
   ↓
Load Balancer
 ↙   ↓   ↘
A    B    C
```

Load Balancer의 목적은 단순히 “공평하게 나누기”가 아니라 다음을 포함합니다.

- 처리량 확장
- 장애 격리
- Backend 교체와 배포
- Health Check
- Routing 정책

---

## 2. L4 Load Balancing

L4는 Transport Layer 정보 중심으로 분산합니다.

주로 보는 정보:

- Source / Destination IP
- Port
- TCP / UDP

장점:

- 비교적 단순하고 빠름
- Application Protocol을 깊게 해석하지 않아도 됨

단점:

- URL Path나 HTTP Header 같은 Application 정보 기반 Routing은 어렵거나 제한적

---

## 3. L7 Load Balancing

L7은 HTTP 같은 Application Layer를 이해하고 분산합니다.

예:

```text
/api/users → Backend Group A
/api/images → Backend Group B
Host: admin.example.com → Admin Backend
```

가능한 기능:

- Path-based Routing
- Host-based Routing
- Header 기반 정책
- TLS Termination
- Authentication / WAF 연계

대신 Protocol Parsing과 추가 처리 비용이 있습니다.

---

## 4. 대표적인 분산 알고리즘

### Round Robin

서버에 순서대로 요청을 보냅니다.

```text
A → B → C → A → B → C
```

단순하지만 요청 처리 시간이 크게 다르면 불균형할 수 있습니다.

### Least Connections

현재 Connection이 적은 서버를 선택합니다.

긴 Connection이 많은 환경에서 유리할 수 있지만 Connection 수가 실제 부하와 완전히 같지는 않습니다.

### Weighted 방식

서버 성능 차이가 있을 때 가중치를 줍니다.

```text
A: weight 3
B: weight 1
```

### Hash 기반

Client IP나 Key를 Hash해 특정 Backend에 매핑할 수 있습니다.

Session affinity나 Cache locality에 도움이 될 수 있습니다.

---

## 5. Health Check

Load Balancer는 Backend가 살아 있는지 확인해야 합니다.

예:

```text
GET /health
```

하지만 단순 Process 생존 여부만 확인하면 부족할 수 있습니다.

- DB 연결 불가
- Thread Pool 고갈
- Disk Full
- Dependency 장애

등을 어느 수준까지 Health Check에 반영할지 설계해야 합니다.

너무 무거운 Health Check 자체가 시스템 부하가 될 수도 있습니다.

---

## 6. Sticky Session

사용자를 특정 Backend에 계속 보내는 방식입니다.

장점:

- In-memory Session 사용이 쉬움

단점:

- 특정 서버에 부하 집중
- 서버 장애 시 Session 유실
- Auto Scaling과 궁합이 나쁠 수 있음

따라서 대규모 서비스에서는 Session을 외부 저장소에 두고 Backend를 Stateless하게 만드는 방향이 자주 선호됩니다.

---

## 7. Load Balancer 자체는 장애 나지 않나?

맞습니다.

Load Balancer가 한 대뿐이면 새로운 SPOF가 됩니다.

실제 환경에서는 다음을 사용합니다.

- Managed Load Balancer
- Active/Standby
- Multiple Zones
- Anycast / DNS 기반 분산

즉 Load Balancing은 Backend만 여러 대 만드는 문제가 아니라 **분산 계층 자체의 가용성**도 고려해야 합니다.

---

## 8. Reverse Proxy와 Load Balancer 관계

둘은 완전히 같은 개념은 아닙니다.

Reverse Proxy는 Server 측 대리자라는 더 넓은 역할이고, Load Balancing은 여러 Backend 중 어디로 보낼지 결정하는 기능입니다.

하나의 제품이 둘 다 수행할 수 있습니다.

```text
Nginx / Envoy
= Reverse Proxy 역할
+ Load Balancing 기능
```

---

## 9. 60초 면접 답변

> Load Balancer는 요청을 여러 Backend에 분산해 처리량과 가용성을 높이는 구성 요소입니다. L4 Load Balancer는 IP와 Port 같은 Transport Layer 정보를 중심으로 분산하고, L7 Load Balancer는 HTTP Host, Path, Header 같은 Application 정보를 이용해 더 정교한 Routing을 할 수 있습니다. Round Robin, Least Connections, Weighted, Hash 기반 정책을 사용할 수 있고, 장애 서버로 트래픽을 보내지 않기 위해 Health Check도 수행합니다. Sticky Session은 편리하지만 확장성과 장애 대응에 제약이 있어 가능하면 Backend를 Stateless하게 만드는 설계가 유리합니다.

---

## 핵심 요약

```text
Load Balancing = Scale + Availability + Routing
L4 = IP / Port 중심
L7 = HTTP 의미까지 이해
Health Check = 장애 Backend 제외
```

다음 주제: CDN과 HTTP Cache
