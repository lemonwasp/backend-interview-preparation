# 12. Observability / SLO

## 한 줄 정의

Observability는 시스템 외부에서 보이는 **Metrics, Logs, Traces**를 이용해 내부 상태와 장애 원인을 추론할 수 있게 만드는 능력이고, SLO는 서비스가 사용자에게 어느 수준의 신뢰성을 제공해야 하는지를 수치로 정의한 목표다.

---

## 1. Monitoring과 Observability의 차이

Monitoring은 주로 "알고 있는 문제"를 감시한다.

예:

- CPU > 90%
- Error Rate > 5%
- Disk 사용량 > 80%

Observability는 한 단계 더 나아간다.

> "왜 느린가? 어디서 병목이 생겼나? 어떤 요청만 실패하나?"

즉 단순 알람보다 **원인을 추론할 수 있는 데이터 구조**가 중요하다.

---

## 2. Three Pillars: Metrics / Logs / Traces

### Metrics

숫자의 시간 변화.

예:

- Request Rate
- Error Rate
- p95 / p99 Latency
- CPU
- Memory
- Queue Depth
- DB Connection Pool Usage

장점: 전체 상태를 빠르게 파악.

### Logs

특정 이벤트의 상세 기록.

예:

```text
order_id=123 payment_failed reason=timeout
```

장점: 개별 사건 분석.

### Traces

한 요청이 여러 서비스를 지나가는 전체 경로.

```text
Client
  -> API Gateway
     -> Order Service
        -> Payment Service
        -> DB
```

장점: 분산 시스템의 latency와 dependency 추적.

---

## 3. RED Method

서비스 관점에서 자주 보는 세 가지:

- **Rate**: 요청량
- **Errors**: 실패율
- **Duration**: latency

예:

```text
RPS = 2,000
Error Rate = 0.4%
p95 = 180ms
p99 = 900ms
```

평균 100ms만 보면 p99 900ms 문제를 놓칠 수 있다.

---

## 4. USE Method

자원 관점에서는:

- **Utilization**: 얼마나 사용 중인가
- **Saturation**: 대기열이 쌓이는가
- **Errors**: 자원 오류가 있는가

예:

DB Connection Pool:

- Utilization: 98%
- Saturation: pool wait 증가
- Errors: connection timeout 증가

---

## 5. SLI / SLO / SLA

### SLI — Service Level Indicator

실제로 측정하는 지표.

예:

- 성공 요청 비율
- 300ms 이하 응답 비율

### SLO — Service Level Objective

목표.

예:

> 월간 요청의 99.9%가 성공해야 한다.

### SLA — Service Level Agreement

고객과의 계약적 약속.

SLA 위반에는 보상이나 계약상 책임이 포함될 수 있다.

즉:

```text
SLI = 측정값
SLO = 내부 목표
SLA = 외부 계약
```

---

## 6. Availability 계산

99.9% availability라고 해보자.

한 달 30일은 약 43,200분이다.

0.1% downtime은 약 43분이다.

따라서 99.9%와 99.99%는 겉보기엔 0.09% 차이지만 허용 장애 시간은 크게 다르다.

높은 SLO는 비용이 증가한다.

- redundancy
- multi-AZ
- faster failover
- 더 많은 운영 인력
- 더 엄격한 배포 절차

---

## 7. Error Budget

SLO가 99.9%라면 0.1%는 허용 가능한 실패 예산이다.

이를 Error Budget이라고 한다.

예:

- 최근 장애로 error budget을 많이 소모했다.
- 새 기능 배포보다 reliability 개선을 우선한다.

즉 Reliability와 개발 속도를 같은 언어로 논의하게 해준다.

---

## 8. 좋은 Alert의 조건

모든 metric에 Alert를 걸면 Alert Fatigue가 생긴다.

좋은 Alert는 보통 사용자 영향과 연결된다.

예:

나쁜 Alert:

```text
CPU 75%
```

더 나은 Alert:

```text
5분 동안 payment success SLI가 SLO burn rate를 초과
```

CPU가 높아도 사용자 영향이 없으면 즉시 장애가 아닐 수 있다.

---

## 9. Burn Rate

Error Budget을 얼마나 빠르게 소비하는지 보는 개념이다.

짧은 시간에 실패율이 매우 높다면 빠른 경보가 필요하다.

반대로 낮은 수준의 지속적 오류는 긴 window에서 감지해야 한다.

그래서 여러 time window를 함께 사용하는 SLO alert가 실용적이다.

---

## 10. Cardinality 문제

Metric label에 무제한 값을 넣으면 비용이 폭증한다.

나쁜 예:

```text
request_latency{user_id="123456789"}
```

사용자 수만큼 time series가 생긴다.

보통 Metric에는 제한된 cardinality의 label을 사용하고, 상세 user/request 정보는 log/trace에 넣는다.

---

## 11. Correlation ID / Trace ID

분산 시스템에서는 하나의 요청이 여러 서비스를 지나간다.

그래서 로그에 공통 식별자를 넣는다.

```text
trace_id=abc123
```

그럼:

```text
Gateway -> Order -> Payment -> DB
```

전체 흐름을 연결해서 볼 수 있다.

---

## 12. Observability도 비용이 있다

모든 요청의 모든 세부 정보를 영원히 저장할 수는 없다.

비용:

- storage
- ingestion
- query cost
- high-cardinality data
- privacy/security

그래서 sampling, retention, aggregation 전략이 필요하다.

---

## 13. System Design 면접에서의 사용

설계를 설명한 뒤 반드시 묻는다.

> "이 시스템이 제대로 동작하는지 어떻게 알죠?"

답변 예:

- API: RPS, error rate, p95/p99
- Queue: depth, oldest message age
- DB: query latency, connection pool wait, replication lag
- Cache: hit ratio, eviction, hot key
- Business: payment success rate

기술 metric과 business SLI를 같이 봐야 한다.

---

## 14. 흔한 오해

### 오해 1: 로그가 많으면 Observability가 좋다

아니다. 검색할 수 있고 상관관계를 추적할 수 있어야 한다.

### 오해 2: 100% availability를 목표로 해야 한다

비용이 매우 크고 대부분의 시스템에 비현실적이다.

### 오해 3: 평균 latency만 보면 된다

Tail latency를 놓친다.

### 오해 4: SLA와 SLO는 같다

아니다. SLA는 외부 계약 성격이 있고 SLO는 내부 운영 목표인 경우가 많다.

---

## 15. 60초 기술면접 답변

> Observability는 Metrics, Logs, Traces를 통해 시스템 내부 상태와 장애 원인을 추론할 수 있게 만드는 능력입니다. 서비스 상태는 RED의 Rate, Error, Duration을 보고 자원은 USE의 Utilization, Saturation, Error를 봅니다. 신뢰성 목표는 SLI로 측정하고 SLO로 목표를 정하며, SLA는 외부 계약입니다. SLO를 100%로 잡기보다 Error Budget을 두고 개발 속도와 Reliability를 균형 있게 운영하는 것이 중요합니다. 또한 평균 latency보다 p95/p99, Queue Depth, DB Pool Wait 같은 saturation 지표를 함께 보고 Trace ID로 여러 서비스의 요청 흐름을 연결합니다.
