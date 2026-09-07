# 21. Rate Limiting

## 왜 필요한가

Rate Limiting은 특정 Client, User, API Key, IP 또는 전체 시스템이 일정 시간 동안 보낼 수 있는 요청량을 제한하는 기술입니다.

목표는 단순히 사용자를 막는 것이 아니라 다음을 보호하는 것입니다.

- 서버 CPU/Memory/Thread Pool
- Database Connection Pool
- 외부 API quota
- 공정한 자원 사용
- Abuse / Bot / accidental traffic spike

## 기본 생각

예를 들어 `100 requests / minute` 정책은 1분 동안 최대 100개까지만 허용한다는 뜻입니다.

하지만 구현 방식에 따라 burst 처리와 정확성이 달라집니다.

## Fixed Window

시간을 고정 구간으로 나눕니다.

```text
12:00:00 ~ 12:00:59 -> 100 requests
12:01:00 ~ 12:01:59 -> 100 requests
```

장점:
- 구현이 단순함
- Counter 하나로 처리 가능

단점:
- 경계 문제
- 12:00:59에 100개 + 12:01:00에 100개가 들어오면 매우 짧은 시간에 200개가 허용될 수 있음

## Sliding Window

현재 시점 기준 최근 N초의 요청을 계산합니다.

Fixed Window보다 실제 요청률을 더 정확히 제어하지만 상태 관리 비용이 더 큽니다.

## Token Bucket

Bucket에 Token이 일정 속도로 채워지고 요청마다 Token을 하나 소비합니다.

```text
refill rate = 10 tokens/sec
bucket capacity = 100
```

평소에는 Token을 모아두었다가 짧은 Burst를 허용할 수 있습니다.

그래서 API Gateway와 네트워크 시스템에서 자주 사용됩니다.

## Leaky Bucket

물이 일정 속도로 빠져나가는 양동이처럼 출력 속도를 일정하게 만듭니다.

Token Bucket이 Burst를 허용하는 데 유리하다면 Leaky Bucket은 traffic smoothing에 더 가깝습니다.

## 어디에서 제한할까?

- Client
- CDN / Edge
- API Gateway
- Reverse Proxy
- Application
- Database 앞의 별도 보호 계층

보통 앞단에서 일찍 차단하는 것이 비싼 downstream 작업을 줄이는 데 유리합니다.

## 무엇을 Key로 잡을까?

예:

```text
IP
User ID
API Key
Tenant ID
Endpoint
User + Endpoint
```

IP 하나만 쓰면 NAT 뒤의 정상 사용자 여러 명을 한 명처럼 취급할 수 있습니다.

## Distributed Rate Limiting

서버가 여러 대라면 각 서버가 독립 Counter를 가지면 전체 제한을 초과할 수 있습니다.

예:

```text
Server A -> 100 req
Server B -> 100 req
```

원래 전체 100개 제한이어도 200개가 통과할 수 있습니다.

그래서 Redis 같은 공유 저장소, centralized limiter 또는 consistent policy가 필요할 수 있습니다.

## 429 Too Many Requests

Rate Limit 초과 시 HTTP에서는 일반적으로 `429 Too Many Requests`를 사용합니다.

필요하면 Client가 언제 재시도할지 알 수 있도록 `Retry-After`를 제공할 수 있습니다.

## Rate Limiting vs Throttling

현업에서는 섞어 쓰기도 하지만 보통:

- Rate Limiting: 일정 요청률을 넘으면 거부
- Throttling: 속도를 늦추거나 처리량을 제어

로 구분할 수 있습니다.

## Backend 면접 연결

질문: "API가 갑자기 10배 트래픽을 받으면 어떻게 보호하겠습니까?"

좋은 답변은 단순히 Scale-out만 말하지 않습니다.

```text
Rate Limit
→ Queue / Backpressure
→ Timeout
→ Circuit Breaker
→ Horizontal Scaling
→ DB / downstream capacity 확인
```

처럼 전체 보호 구조를 설명하는 것이 좋습니다.

## 흔한 오해

### "Rate Limiting은 DDoS 방어다"

일부 도움은 되지만 대규모 DDoS는 네트워크/Edge/WAF/CDN 수준의 방어가 필요합니다.

### "Distributed 환경에서는 Redis Counter만 쓰면 끝이다"

Redis 자체 장애, hot key, latency, consistency trade-off를 고려해야 합니다.

## 60초 면접 답변

Rate Limiting은 일정 시간 동안 Client나 User가 보낼 수 있는 요청 수를 제한해 시스템 자원을 보호하는 기술입니다. Fixed Window는 단순하지만 경계 Burst 문제가 있고, Sliding Window는 더 정확하지만 상태 관리 비용이 큽니다. Token Bucket은 일정 속도로 Token을 보충하면서 Bucket 용량만큼 Burst를 허용할 수 있어 실무에서 자주 사용됩니다. 분산 서버에서는 각 인스턴스가 독립 Counter를 가지면 전체 제한을 지킬 수 없으므로 공유 저장소나 중앙 Limiter가 필요할 수 있습니다. HTTP에서는 보통 제한 초과에 429를 사용합니다.