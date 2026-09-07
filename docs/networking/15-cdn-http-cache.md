# 15. CDN과 HTTP Cache

## 학습 목표

- CDN이 왜 필요한지 설명할 수 있다.
- Browser Cache, Proxy Cache, CDN Cache를 구분할 수 있다.
- `Cache-Control`, `ETag`, `Last-Modified`의 역할을 설명할 수 있다.
- Freshness와 Validation을 구분할 수 있다.
- Cache Invalidation이 어려운 이유를 설명할 수 있다.

---

## 1. CDN은 무엇인가?

CDN(Content Delivery Network)은 사용자와 가까운 Edge 위치에서 콘텐츠를 제공해 Origin Server까지의 요청을 줄이고 Latency를 낮추는 분산 네트워크입니다.

```text
User
  ↓
Nearby CDN Edge
  ↓ cache miss
Origin Server
```

CDN의 핵심 효과:

- Latency 감소
- Origin 부하 감소
- 대역폭 절감
- 대규모 정적 콘텐츠 배포
- 장애·공격 완화에 도움

---

## 2. Cache는 어디에 있을 수 있나?

HTTP Cache는 여러 계층에 존재할 수 있습니다.

```text
Browser Cache
   ↓
Corporate / Proxy Cache
   ↓
CDN Edge Cache
   ↓
Reverse Proxy Cache
   ↓
Origin
```

따라서 “캐시가 있다”는 말만으로는 부족합니다.

> 어느 계층의 캐시이며, 누가 공유하고, 언제 만료되는가?

를 봐야 합니다.

---

## 3. Freshness와 Validation

HTTP Cache는 크게 두 가지 방식으로 Origin 요청을 줄일 수 있습니다.

### Freshness

아직 신선하다고 판단하면 Origin에 묻지 않고 Cached Response를 바로 사용합니다.

예:

```http
Cache-Control: max-age=3600
```

### Validation

만료되었더라도 전체 Content를 다시 받을 필요가 있는지 확인할 수 있습니다.

```http
If-None-Match: "abc123"
```

Origin이 변경되지 않았다고 판단하면:

```http
304 Not Modified
```

를 반환할 수 있습니다.

---

## 4. Cache-Control

대표 Directive:

### `max-age`

Client Cache에서 얼마나 오래 Fresh한지 지정합니다.

### `s-maxage`

Shared Cache에서 별도의 Freshness 시간을 지정할 수 있습니다.

### `no-cache`

이름 때문에 오해하기 쉽습니다.

`no-cache`는 “절대 저장하지 마라”가 아니라 보통 **사용 전에 재검증하라**는 의미입니다.

### `no-store`

응답을 저장하지 않도록 요구합니다.

### `private`

Shared Cache가 아니라 특정 사용자 Cache에만 저장되어야 하는 응답에 사용합니다.

### `public`

Shared Cache가 저장할 수 있음을 명시하는 데 사용할 수 있습니다.

---

## 5. ETag와 Last-Modified

### ETag

Resource Version을 식별하는 값입니다.

```http
ETag: "v123"
```

Client는 다음 요청에서:

```http
If-None-Match: "v123"
```

를 보낼 수 있습니다.

### Last-Modified

마지막 변경 시각을 사용합니다.

```http
Last-Modified: Tue, 08 Sep 2026 12:00:00 GMT
```

Client는:

```http
If-Modified-Since: ...
```

를 사용할 수 있습니다.

ETag는 시간보다 더 정밀한 Version Identifier로 사용할 수 있지만 생성 비용과 분산 환경의 일관성도 고려해야 합니다.

---

## 6. Cache Key

CDN이 무엇을 같은 Resource로 볼지는 Cache Key에 달려 있습니다.

일반적으로 다음이 영향을 줄 수 있습니다.

- Scheme
- Host
- Path
- Query String
- 일부 Header

잘못된 Cache Key 설계는 위험합니다.

예를 들어 사용자별 인증 응답을 Authorization Header를 무시하고 같은 Cache Entry로 공유하면 다른 사용자의 데이터가 노출될 수 있습니다.

따라서 Personalized Response는 Shared Cache 정책을 매우 신중하게 설계해야 합니다.

---

## 7. Cache Invalidation

Cache의 대표적인 어려움입니다.

Origin 데이터는 바뀌었는데 Edge에는 예전 데이터가 남아 있을 수 있습니다.

대응 방법:

- 짧은 TTL
- Purge / Invalidation API
- Versioned URL
- Content Hash filename

예:

```text
/app.js          ← 변경 시 stale 문제 가능
/app.a8f93c.js   ← 내용이 바뀌면 URL 자체 변경
```

Static Asset에는 Content Hash 방식이 매우 강력합니다.

---

## 8. Cache Stampede

인기 Cache Entry가 동시에 만료되면 많은 Request가 Origin으로 몰릴 수 있습니다.

```text
10000 clients
   ↓ cache expires
10000 origin requests
```

이를 Cache Stampede 또는 Thundering Herd 문제로 볼 수 있습니다.

대응:

- Request Coalescing
- Stale-while-revalidate
- TTL Jitter
- Background Refresh

---

## 9. CDN과 동적 API

CDN은 이미지·JS·CSS만 위한 것은 아닙니다.

적절한 Cache Key와 TTL을 설계하면 일부 API Response도 Edge Cache할 수 있습니다.

하지만 다음은 주의해야 합니다.

- 사용자별 데이터
- 인증 정보
- 매우 빠르게 바뀌는 데이터
- 개인정보
- 권한에 따라 달라지는 Response

Caching은 성능 기술이면서 동시에 데이터 일관성과 보안 정책입니다.

---

## 10. 60초 면접 답변

> CDN은 사용자와 가까운 Edge에서 콘텐츠를 캐시해 Latency와 Origin 부하를 줄이는 분산 네트워크입니다. HTTP Cache에서는 `Cache-Control`로 Freshness 정책을 지정하고, 만료 후에는 ETag나 Last-Modified를 이용해 조건부 요청을 보내 304 응답으로 전체 Body 전송을 피할 수 있습니다. Cache Key와 Shared Cache 정책을 잘못 설계하면 사용자별 데이터가 섞일 수 있으므로 특히 인증된 API는 주의해야 합니다. 또한 Cache Invalidation과 동시에 만료될 때 발생하는 Cache Stampede도 중요한 운영 문제입니다.

---

## 핵심 요약

```text
CDN = Edge delivery + Shared cache
Freshness = Origin에 묻지 않고 사용
Validation = 변경 여부만 확인
Cache-Control = 저장/재사용 정책
ETag = Resource version
```

다음 주제: WebSocket과 gRPC
