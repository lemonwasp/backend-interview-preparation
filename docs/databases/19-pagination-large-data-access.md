# 19. Pagination / Large Data Access

## 한 줄 핵심

Pagination은 단순 UI 기법이 아니라 DB가 얼마나 많은 Row를 읽고 버릴지 결정하는 Query 설계 문제이며, 큰 OFFSET은 뒤로 갈수록 비용이 커질 수 있다.

## OFFSET Pagination

```sql
SELECT id, created_at, title
FROM posts
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 100000;
```

DB는 보통 앞의 많은 Row를 찾아 순서를 결정한 뒤 상당 부분을 건너뛰어야 한다.

따라서 페이지 번호가 깊어질수록:

- 읽는 Row 수 증가
- CPU/I/O 증가
- latency 증가

할 수 있다.

## Keyset / Cursor Pagination

마지막으로 본 Row의 정렬 Key를 기억한다.

```sql
SELECT id, created_at, title
FROM posts
WHERE (created_at, id) < (?, ?)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

적절한 Composite Index가 있으면 다음 구간부터 바로 탐색하기 쉽다.

## Stable Ordering

`created_at` 값이 같은 Row가 많다면 이것만으로 정렬하면 페이지 사이에서 중복/누락이 생길 수 있다.

그래서 보통 unique tie-breaker를 붙인다.

```text
ORDER BY created_at DESC, id DESC
```

Cursor에도 두 값을 함께 넣는다.

## OFFSET의 장점

- 임의 페이지 번호로 이동하기 쉬움
- 구현과 이해가 간단함
- 작은 Dataset에서는 충분히 실용적

따라서 OFFSET이 무조건 잘못된 것은 아니다.

## Keyset의 trade-off

장점:

- 깊은 페이지에서도 비교적 일정한 비용
- 실시간으로 Row가 추가되는 Feed에 유리

단점:

- 임의의 5000페이지로 바로 이동하기 어려움
- Cursor 설계 필요
- 정렬 조건과 Index가 중요

## Large Result Set

수백만 Row를 Application Memory에 한 번에 올리면:

- App memory pressure
- GC pressure
- DB Connection 장시간 점유
- Network bandwidth 증가

가 발생한다.

대용량 Batch 작업은 보통:

- Chunk/Batch 단위 조회
- Streaming
- Keyset 기반 iteration
- 필요한 Column만 Projection

을 고려한다.

## `SELECT *`의 비용

필요하지 않은 큰 TEXT/BLOB Column까지 읽으면:

- 더 많은 Page I/O
- Network 전송량 증가
- ORM materialization 비용 증가
- Covering Index 사용 기회 감소

할 수 있다.

## Count(*) 문제

매 페이지마다 거대한 Dataset의 정확한 전체 Count를 계산하는 것이 비쌀 수 있다.

제품 요구사항에 따라:

- 정확한 Count
- Approximate Count
- `hasNext`만 제공

중 어떤 것이 실제로 필요한지 구분한다.

## Pagination과 동시 변경

사용자가 1페이지를 본 뒤 새 Row가 삽입되면 OFFSET 기반 다음 페이지의 위치가 밀릴 수 있다.

Feed처럼 변화가 많은 데이터에서는 Keyset 방식이 더 자연스러운 경우가 많다.

## Index 예시

Query:

```sql
WHERE tenant_id = ?
  AND (created_at, id) < (?, ?)
ORDER BY created_at DESC, id DESC
LIMIT 20
```

고려할 수 있는 Index:

```text
(tenant_id, created_at, id)
```

실제 최적 Index는 DBMS, sort direction, selectivity, query pattern에 따라 EXPLAIN으로 검증한다.

## 60초 면접 답변

Pagination은 DB 관점에서는 얼마나 많은 Row를 읽고 버리는지의 문제입니다. OFFSET 방식은 간단하고 임의 페이지 이동이 쉽지만 깊은 페이지로 갈수록 앞 Row를 많이 처리해야 해서 비용이 커질 수 있습니다. Keyset Pagination은 마지막 Row의 정렬 Key를 Cursor로 저장하고 그 다음 범위부터 조회하기 때문에 적절한 Index가 있으면 깊이에 따른 비용 증가를 줄일 수 있습니다. 대신 임의 페이지 이동이 어렵고 안정적인 정렬을 위해 created_at과 unique id 같은 tie-breaker를 함께 써야 합니다. 대용량 조회에서는 Pagination뿐 아니라 Projection, Batch, Streaming으로 App memory와 Connection 점유 시간도 같이 줄여야 합니다.
