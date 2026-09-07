# 13. Graceful Degradation

## 한 줄 정의

Graceful Degradation은 일부 dependency나 기능이 실패해도 **핵심 사용자 경로는 가능한 범위에서 계속 제공하도록 시스템 기능을 단계적으로 축소하는 설계**다.

---

## 1. 쉬운 비유

비행기에서 기내 Wi-Fi가 고장 났다고 비행 자체를 취소하지는 않는다.

핵심 기능은 비행이고 Wi-Fi는 부가 기능이다.

백엔드도 마찬가지다.

추천 서비스가 죽었다고 상품 상세 페이지 전체를 500으로 만들 필요는 없다.

```text
핵심 기능: 상품 조회
부가 기능: 추천 상품
```

추천이 실패하면 추천 영역만 비우고 상품 조회는 계속 제공할 수 있다.

---

## 2. 왜 필요한가

분산 시스템은 dependency가 많다.

```text
API
├── User Service
├── Recommendation Service
├── Payment Service
├── Cache
└── DB
```

모든 dependency를 "반드시 성공"으로 취급하면 한 서비스 장애가 전체 장애로 확대된다.

그래서 기능을 다음처럼 분류할 필요가 있다.

- Critical
- Important
- Optional

---

## 3. 핵심 기법 1: Fallback

예:

```text
Recommendation Service timeout
        ↓
최근 인기 상품 목록 반환
```

또는:

```text
Profile image service failure
        ↓
Default avatar
```

Fallback은 실패를 숨기는 것이 아니라 **허용 가능한 대체 동작**을 정의하는 것이다.

---

## 4. 핵심 기법 2: Stale Data 사용

최신 데이터가 아니어도 되는 경우 오래된 Cache를 임시로 사용할 수 있다.

```text
Origin unavailable
    ↓
serve stale cache
```

예:

- 상품 설명
- 뉴스 목록
- 추천 결과

하지만 다음 데이터에는 위험하다.

- 계좌 잔액
- 주문 결제 상태
- 재고 확정

즉 데이터 성격별로 허용 가능한 staleness를 정해야 한다.

---

## 5. 핵심 기법 3: Feature Shedding

과부하 시 비핵심 기능을 끈다.

예:

```text
정상 상태:
상품 + 추천 + 리뷰 요약 + 실시간 인기 순위

과부하 상태:
상품 + 기본 리뷰만
```

CPU나 DB가 포화되기 전에 expensive optional path를 줄여 핵심 트래픽을 보호한다.

---

## 6. 핵심 기법 4: Load Shedding

처리 용량을 넘는 요청을 전부 queue에 쌓으면 시스템 전체가 느려질 수 있다.

때로는 일부 요청을 빠르게 거절하는 것이 낫다.

예:

```text
HTTP 503
Retry-After
```

Load Shedding의 목적은 "모든 요청을 늦게 실패"시키는 대신 "일부 요청을 빠르게 실패"시켜 전체 시스템을 보호하는 것이다.

---

## 7. Circuit Breaker와 연결

반복적으로 실패하는 dependency에 계속 요청을 보내면:

- Thread/Connection 소비
- Timeout 누적
- Retry Storm
- Latency 증가

Circuit Breaker가 Open되면 해당 호출을 빠르게 실패시키고 fallback으로 전환할 수 있다.

```text
dependency unhealthy
    ↓
Circuit Open
    ↓
Fallback
```

---

## 8. Retry와 Degradation

Retry는 일시 장애에는 도움이 되지만 무제한 retry는 위험하다.

따라서 일반적인 순서는:

1. timeout
2. 제한된 retry + backoff/jitter
3. circuit breaker
4. fallback/degradation

상황에 따라 retry 없이 바로 fallback하는 것이 더 낫기도 한다.

---

## 9. Queue 기반 Degradation

동기 처리 대신 나중에 처리할 수 있다면 queue로 밀어낼 수 있다.

예:

- 이메일 발송
- 로그 분석
- 알림
- 썸네일 생성

```text
Request
  ↓
DB commit
  ↓
Queue
  ↓
Background Worker
```

핵심 요청 latency를 줄이고 dependency failure를 격리할 수 있다.

---

## 10. Partial Response

API 일부 데이터만 없어도 응답할 수 있다면 partial response를 고려할 수 있다.

예:

```json
{
  "product": { "id": 1, "name": "A" },
  "recommendations": null
}
```

단, API 계약에서 어떤 필드가 optional인지 명확해야 한다.

---

## 11. UX와 연결

Graceful Degradation은 Backend만의 문제가 아니다.

사용자에게 다음을 알려야 할 수 있다.

- "추천 정보를 불러오지 못했습니다."
- "실시간 순위 대신 최근 데이터를 표시합니다."
- "결제 확인 중입니다. 잠시 후 주문 상태를 확인해주세요."

특히 eventual consistency와 결합되면 중간 상태 UX가 중요하다.

---

## 12. 우선순위 기반 처리

모든 트래픽의 가치가 같지 않을 수 있다.

예:

- 결제: 최우선
- 상품 조회: 높음
- 추천 refresh: 낮음
- analytics export: 매우 낮음

과부하 시 낮은 우선순위 작업을 줄여 핵심 경로를 보호할 수 있다.

---

## 13. 흔한 오해

### 오해 1: Degradation은 오류를 숨기는 것이다

아니다. 핵심 기능과 부가 기능의 우선순위를 설계하는 것이다.

### 오해 2: stale cache는 항상 안전하다

아니다. 금융/결제/재고처럼 최신성이 중요한 데이터에는 위험하다.

### 오해 3: Retry를 많이 하면 복구된다

오히려 downstream을 더 공격해 장애를 키울 수 있다.

### 오해 4: 모든 기능을 계속 제공해야 availability가 높다

핵심 기능을 유지하기 위해 일부 기능을 버리는 것이 전체 availability를 높일 수 있다.

---

## 14. 면접 설계 패턴

면접에서 장애 대응을 설명할 때:

1. Critical path를 정한다.
2. Optional dependency를 찾는다.
3. timeout/retry/circuit breaker를 설정한다.
4. fallback/stale data/partial response를 정의한다.
5. 과부하 시 load shedding/feature shedding을 고려한다.
6. 사용자에게 중간 상태를 어떻게 표현할지 말한다.

---

## 15. 60초 기술면접 답변

> Graceful Degradation은 일부 dependency가 실패해도 핵심 기능을 유지하기 위해 부가 기능을 단계적으로 줄이는 설계입니다. 먼저 critical path와 optional dependency를 구분하고, dependency에는 timeout과 제한된 retry, circuit breaker를 둡니다. 실패 시에는 stale cache, default value, partial response 같은 fallback을 사용하고 과부하 상황에서는 feature shedding이나 load shedding으로 핵심 트래픽을 보호합니다. 다만 stale data가 안전한지는 데이터 성격에 따라 다르며 결제나 재고처럼 최신성이 중요한 데이터에는 적용하면 안 됩니다.
