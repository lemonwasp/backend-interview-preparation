# 05. Load Balancing and Horizontal Scaling

Load Balancing의 핵심은 **트래픽을 여러 instance에 분산하고, 장애 난 instance를 우회하며, scale-out을 가능하게 만드는 것**이다.

## 1. 비유

마트 계산대가 하나뿐이면 손님이 몰릴 때 줄이 길어진다.

계산대를 여러 개 열고 직원이 손님을 적절한 줄로 안내하면 처리량이 늘어난다.

Backend에서는:

```text
Client
  ↓
Load Balancer
  ↓
App 1 / App 2 / App 3
```

## 2. Vertical vs Horizontal Scaling

### Vertical Scaling
한 서버를 더 강하게 만든다.
- CPU 증가
- RAM 증가

장점:
- 단순함

단점:
- 상한 존재
- 큰 instance 비용 증가
- single-machine failure risk

### Horizontal Scaling
서버 수를 늘린다.

장점:
- scale-out 가능
- redundancy 확보

단점:
- distributed system 복잡도 증가
- statelessness, coordination 필요

## 3. Load Balancer의 주요 역할

- traffic distribution
- health check
- failed instance 제외
- TLS termination 가능
- connection/request routing

Application Gateway/API Gateway와 역할이 겹칠 수 있지만 개념적으로는 traffic distribution이 핵심이다.

## 4. L4 vs L7 Load Balancing

### L4
IP/Port/TCP/UDP 수준에서 분산한다.

장점:
- 단순하고 빠름

### L7
HTTP path/header/host 같은 application 정보를 보고 분산한다.

예:
```text
/api/images → Image Service
/api/users  → User Service
```

장점:
- 세밀한 routing

비용:
- application protocol 처리 복잡도

## 5. Routing Algorithm

대표:
- Round Robin
- Weighted Round Robin
- Least Connections
- Hash-based routing

어떤 방식이 좋은지는 workload에 따라 다르다.

요청 시간이 크게 다른 서비스에서는 단순 Round Robin이 균등한 실제 부하를 보장하지 않을 수 있다.

## 6. Health Check

Load Balancer가 instance 생존 여부를 판단해야 한다.

### Liveness
프로세스 자체가 살아 있는가.

### Readiness
현재 traffic을 받을 준비가 됐는가.

예:
- startup 중
- DB 연결 실패
- dependency 준비 안 됨

이런 instance는 살아 있어도 readiness는 false일 수 있다.

## 7. Stateless Service와 연결

App이 stateless하면 어느 instance로 요청을 보내도 된다.

그래서:

```text
Stateless App
+ Load Balancer
+ Horizontal Scaling
```

조합이 자연스럽다.

Stateful session 때문에 특정 서버에 붙어야 하면 sticky session이 필요할 수 있고 scaling 유연성이 낮아진다.

## 8. Auto Scaling

지표를 보고 instance 수를 자동 조절할 수 있다.

예:
- CPU
- RPS
- queue depth
- latency
- custom metric

CPU 하나만 보면 충분하지 않을 수 있다.

예를 들어 I/O-bound API는 CPU가 낮아도 connection pool이나 downstream latency 때문에 이미 포화일 수 있다.

## 9. Scale-out이 모든 병목을 해결하지 않는다

App instance를 10배 늘려도 DB capacity가 그대로라면:

```text
App 10 → DB
App 20 → DB
App 50 → DB overload
```

특히 DB connection pool은:

```text
instance count × pool size
```

로 증가한다.

그래서 app scale-out은 downstream budget과 함께 봐야 한다.

## 10. Load Balancer 자체의 HA

Load Balancer가 single instance라면 SPOF가 된다.

실제 managed LB나 HAProxy/Nginx cluster 등은 LB 계층 자체도 redundancy를 고려한다.

"LB를 넣었으니 HA"가 자동으로 성립하는 것은 아니다.

## 11. Connection Draining

배포나 scale-in 때 instance를 즉시 죽이면 진행 중 요청이 끊길 수 있다.

보통:
1. 새 요청을 더 이상 받지 않음
2. 기존 요청/connection을 일정 시간 처리
3. 종료

이를 connection draining/graceful shutdown 관점으로 본다.

## 12. Sticky Session

특정 user를 같은 backend로 routing한다.

장점:
- in-memory session 유지 가능

단점:
- load imbalance
- failure recovery 불리
- autoscaling 유연성 감소

가능하면 shared state/stateless architecture가 일반적으로 더 유연하다.

## 13. Load Balancer와 Cache/Queue

System Design에서는 계층을 따로 보지 말고 함께 본다.

예:

```text
Client
  ↓
CDN
  ↓
Load Balancer
  ↓
Stateless App Instances
  ↓          ↓
Cache       Queue
  ↓          ↓
DB         Workers
```

각 계층이 다른 병목을 해결한다.

## 14. 장애 시나리오

### App instance 하나 죽음
Health check가 감지하고 routing에서 제외한다.

### 특정 instance만 느림
Least-connections만으로 완벽히 해결되지 않을 수 있다. latency-aware metric, health threshold, autoscaling 등을 고려한다.

### 전체 app은 건강하지만 DB가 느림
App scale-out만 하면 오히려 DB pressure가 커질 수 있다.

### Deployment
Readiness를 끄고 drain 후 종료해야 in-flight request 손실을 줄일 수 있다.

## 15. 흔한 오해

### Horizontal Scaling = 무한 확장
아니다. DB, cache, network, external API가 병목이 된다.

### Load Balancer = 모든 장애 해결
아니다. LB 뒤의 shared dependency가 죽으면 전체 서비스가 실패할 수 있다.

### CPU만 보면 autoscaling 가능
아니다. workload에 따라 queue depth, latency, concurrency가 더 중요할 수 있다.

## 16. 60초 면접 답변

> Load Balancer는 여러 backend instance에 traffic을 분산하고 health check를 통해 장애 instance를 제외해 horizontal scaling과 availability를 지원합니다. L4는 transport 수준, L7은 HTTP path나 header 같은 application 정보를 보고 routing합니다. Stateless app과 조합하면 instance를 자유롭게 추가·제거하기 쉬워집니다. 하지만 app을 scale-out한다고 전체 시스템이 자동으로 확장되는 것은 아닙니다. instance 수가 늘면 DB connection과 downstream load도 같이 증가하므로 DB, cache, queue 같은 shared dependency capacity를 함께 봐야 합니다. 또 배포나 scale-in 시에는 readiness와 connection draining을 사용해 in-flight request를 안전하게 처리해야 합니다.
