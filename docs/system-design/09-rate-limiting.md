# 09. Rate Limiting

## 한 줄 정의

Rate Limiting은 **한 사용자·토큰·IP·API·Tenant 등이 일정 시간 동안 사용할 수 있는 요청량을 제한해 시스템과 다른 사용자를 보호하는 설계**다.

## 왜 필요한가

Rate Limit은 단순 보안 기능이 아니다.

다음을 보호한다.

- API 서버 CPU / ThreadPool
- DB Connection / Query capacity
- 외부 API quota
- 비용이 큰 AI/검색/파일 처리
- 다른 정상 사용자

즉 `과도한 요청을 429로 막는다`보다 **어떤 자원이 병목인지에 맞춰 제한 기준을 설계하는 것**이 중요하다.

---

## 1. 무엇을 기준으로 제한할까

대표 Key:

- IP
- User ID
- API Key
- Tenant ID
- Endpoint
- Global limit

IP만 쓰면 NAT 뒤의 정상 사용자를 함께 제한할 수 있고, User ID만 쓰면 로그인 전 공격을 막기 어렵다.

실무에서는 여러 계층을 조합할 수 있다.

---

## 2. Fixed Window

예: 1분에 100회.

```text
12:00:00 ~ 12:00:59 -> 100
12:01:00 ~ 12:01:59 -> 100
```

장점:

- 구현 단순
- 저장 비용 낮음

단점:

- window 경계에서 burst가 가능

예를 들어 12:00:59에 100회, 12:01:00에 100회가 몰릴 수 있다.

---

## 3. Sliding Window

최근 60초처럼 실제 이동 구간을 기준으로 본다.

장점:

- Fixed Window보다 경계 burst를 줄임

단점:

- 정확하게 구현하면 storage/계산 비용이 커질 수 있음

실제로는 approximate sliding window를 사용하기도 한다.

---

## 4. Token Bucket

Bucket에 token이 일정 속도로 채워지고 요청마다 token을 소비한다.

```text
capacity = 100 tokens
refill = 10 tokens/sec
```

특징:

- 평균 rate를 제한하면서 일정 burst 허용
- API rate limiting에 자주 적합

### 핵심

Bucket capacity는 허용 burst 크기이고, refill rate는 장기 평균 허용률이다.

---

## 5. Leaky Bucket

물이 일정 속도로 빠져나가듯 요청을 일정 속도로 처리하도록 평탄화한다.

Token Bucket이 burst 허용에 더 자연스럽다면, Leaky Bucket은 output rate smoothing 관점으로 이해할 수 있다.

---

## 6. Distributed Rate Limiter

App Instance가 여러 개라면 각 Instance가 별도 Counter를 가지면 전체 limit이 깨질 수 있다.

예:

```text
Instance A: 100
Instance B: 100
Instance C: 100
```

원래 전체 제한이 100인데 실제 300이 가능해질 수 있다.

대안:

- Redis 같은 shared store
- centralized gateway
- consistent partitioning
- local + global hybrid limit

---

## 7. 정확성과 Latency의 Trade-off

모든 요청마다 remote Redis round trip을 하면 limiter 자체가 latency와 dependency가 된다.

대안:

- local token cache
- batching
- approximate counters
- hierarchical limits

Rate Limiter도 scale해야 한다.

---

## 8. 429와 Retry-After

제한을 초과하면 일반적으로 HTTP `429 Too Many Requests`를 사용할 수 있다.

가능하면 client에게:

- Retry-After
- remaining quota
- reset time

같은 정보를 제공할 수 있다.

단, 공개 API에서는 내부 quota 정보를 얼마나 노출할지도 정책 문제다.

---

## 9. Rate Limit vs Backpressure

### Rate Limit

입력 요청량 자체를 정책적으로 제한한다.

### Backpressure

downstream 처리 능력에 맞춰 producer 속도를 늦추거나 queue를 제한한다.

둘은 함께 사용할 수 있다.

예:

```text
Client Rate Limit
     ↓
API
     ↓
Bounded Queue / Backpressure
     ↓
DB
```

---

## 10. Rate Limit vs Load Shedding

### Rate Limit

미리 정한 quota 중심.

### Load Shedding

시스템이 실제 과부하 상태일 때 일부 요청을 버려 핵심 기능을 살린다.

예:

- CPU > threshold
- queue depth 급증
- dependency timeout 폭증

둘 다 보호 장치지만 트리거가 다르다.

---

## 11. Multi-tenant Fairness

한 대형 Tenant가 전체 자원을 독점하면 다른 고객이 피해를 본다.

따라서:

- per-tenant limit
- weighted quota
- premium tier
- endpoint별 cost

를 고려할 수 있다.

요청 1개가 항상 같은 비용이라는 가정도 위험하다.

---

## 12. 흔한 오해

### 오해 1: Rate Limit은 DDoS 방어만 위한 것이다

아니다. 비용, quota, fairness, downstream 보호에도 중요하다.

### 오해 2: IP 기준이면 충분하다

NAT, proxy, bot rotation 때문에 한계가 있다.

### 오해 3: 각 App Instance에서 100회 제한하면 전체도 100회다

아니다. distributed coordination이 없으면 instance 수만큼 늘어날 수 있다.

### 오해 4: Limiter는 절대로 실패하면 안 된다

Limiter 자체도 dependency다. 장애 시 fail-open/fail-closed 정책을 기능 위험도에 따라 정해야 한다.

---

## Fail-open vs Fail-closed

Rate Limit store가 죽었다고 하자.

### Fail-open

요청을 통과시킨다.

- availability 유리
- abuse/cost 위험

### Fail-closed

요청을 막는다.

- 보호 강함
- 정상 사용자까지 차단 가능

예:

- 무료 검색 API: fail-open 가능성
- 고비용 결제/AI quota: 더 보수적 정책 가능

---

## 60초 면접 답변

> Rate Limiting은 사용자나 API Key, Tenant 같은 기준으로 요청량을 제한해 서버와 downstream 자원을 보호하는 설계입니다. 대표 알고리즘은 Fixed Window, Sliding Window, Token Bucket, Leaky Bucket이고, Token Bucket은 평균 rate를 제한하면서 일정 burst를 허용할 수 있습니다. 여러 App Instance가 있다면 local counter만으로는 전체 limit이 깨지므로 shared store나 gateway 같은 distributed coordination이 필요합니다. 또한 limiter 자체의 latency와 장애도 고려해야 하고, 장애 시 fail-open과 fail-closed 정책을 기능의 위험도에 맞춰 선택합니다. Rate Limit은 quota 정책이고 Backpressure나 Load Shedding과는 목적과 트리거가 다릅니다.