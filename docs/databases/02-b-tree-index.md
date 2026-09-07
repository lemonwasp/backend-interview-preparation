# 02. B-Tree Index

## 한 줄 정의

B-Tree 계열 Index는 정렬된 Key를 여러 단계의 Page/Node에 나눠 저장해, 전체 Table을 처음부터 끝까지 읽지 않고도 원하는 위치를 빠르게 찾게 해주는 자료구조다.

---

## 1. Index가 없으면 어떻게 찾을까?

```sql
SELECT * FROM users WHERE email = 'a@example.com';
```

email에 Index가 없다면 DB는 많은 경우 Table의 Row를 차례로 확인해야 한다.

이것이 Full Table Scan이다.

Row가 100개라면 큰 문제가 아닐 수 있지만 수천만 건이라면 비용이 커진다.

Index는 책의 색인처럼 `어디에 있는지 찾는 구조`를 별도로 유지한다.

---

## 2. 왜 Binary Search Tree가 아니라 B-Tree 계열일까?

DB의 핵심 저장 단위는 CPU register가 아니라 보통 **Page**다.

Storage나 Buffer Pool에서 Page를 읽는 비용은 단순한 메모리 비교보다 훨씬 크다.

따라서 Tree 높이를 최대한 낮추는 것이 중요하다.

B-Tree 계열은 Node 하나에 많은 Key와 Child Pointer를 저장한다.

```text
                [20 | 40 | 70]
              /      |      |      \
         <20      20~40   40~70    >70
```

Branching Factor가 크기 때문에 수백만~수십억 건이어도 Tree 높이가 비교적 낮게 유지된다.

실제 구현은 DBMS마다 B-Tree, B+Tree 변형 등 차이가 있다.

---

## 3. 탐색 흐름

예를 들어 `id = 42`를 찾는다면:

1. Root Page를 읽는다.
2. Key 범위를 보고 적절한 Child Page를 선택한다.
3. Intermediate Page를 따라간다.
4. Leaf Page에서 42를 찾는다.
5. 필요한 Row 또는 Row 위치를 얻는다.

즉 전체 Row 수가 늘어나도 매번 전부 읽을 필요가 없다.

---

## 4. Equality Search와 Range Search

B-Tree 계열의 큰 장점은 정렬 상태다.

### Equality

```sql
WHERE id = 42
```

### Range

```sql
WHERE created_at >= '2026-09-01'
  AND created_at < '2026-10-01'
```

시작 위치를 찾은 뒤 인접한 Leaf를 순서대로 읽을 수 있어 Range Query에도 강하다.

---

## 5. Index가 공짜가 아닌 이유

Index를 만들면 별도 자료구조를 유지해야 한다.

### 장점
- SELECT 검색을 크게 줄일 수 있다.
- 정렬/범위 조건에 도움을 줄 수 있다.
- Join Key 탐색을 빠르게 할 수 있다.

### 비용
- Storage 사용 증가
- INSERT 시 Index에도 Key 추가
- DELETE 시 Index Entry 제거
- UPDATE 대상 Column이 Index에 포함되면 Index 수정
- Page Split 및 Maintenance 비용

따라서 모든 Column에 Index를 만들면 오히려 쓰기 비용과 저장 비용이 커진다.

---

## 6. Page Split

정렬된 Leaf Page가 꽉 찬 상태에서 중간 위치에 새 Key를 넣어야 할 수 있다.

그러면 DB는 Page를 나누고 Tree 구조를 조정할 수 있다.

이것이 Page Split이다.

Random한 Key 삽입이 많으면 Page locality와 공간 사용에 불리할 수 있다.

이 때문에 Primary Key의 값 분포와 생성 방식이 Index 성능에 영향을 줄 수 있다.

---

## 7. Composite Index

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at);
```

이 Index는 단순히 두 개의 독립 Index가 아니다.

정렬 기준은 개념적으로:

```text
(user_id, created_at)
```

순서다.

따라서 보통 다음 Query에 유리하다.

```sql
WHERE user_id = ?
```

또는

```sql
WHERE user_id = ?
  AND created_at >= ?
```

하지만 `created_at`만 조건으로 사용할 때 동일한 효율을 기대하면 안 된다.

이 개념을 흔히 Leftmost Prefix 특성으로 설명한다.

구체적인 Optimizer 동작은 DBMS마다 다를 수 있다.

---

## 8. Selectivity

Selectivity는 조건이 얼마나 적은 Row를 골라내는지와 관련된 개념이다.

예:

```text
status = 'ACTIVE'
```

전체 1억 건 중 9천만 건이 ACTIVE라면 Index를 타더라도 엄청난 Row를 읽어야 한다.

반면:

```text
email = 'unique@example.com'
```

결과가 한 건이라면 Index 가치가 높다.

따라서 `Index가 존재한다 = DB가 반드시 사용한다`가 아니다.

Optimizer는 비용을 비교해 Table Scan이 더 싸다고 판단할 수도 있다.

---

## 9. Covering Index

Query에 필요한 Column이 전부 Index 안에 있다면 Table 본문을 추가로 읽지 않고 Index만으로 결과를 만들 수 있는 경우가 있다.

예:

```sql
CREATE INDEX idx_user_email_name
ON users(email, name);

SELECT name
FROM users
WHERE email = ?;
```

DBMS와 실행 계획에 따라 Index-only Scan 또는 Covering Index 효과를 얻을 수 있다.

하지만 Index가 넓어질수록 Storage와 Write Cost도 커진다.

---

## 10. Index가 있어도 느릴 수 있는 이유

- 낮은 Selectivity
- 너무 많은 Row 반환
- Composite Index Column 순서가 Query와 맞지 않음
- 함수/형 변환 때문에 Index 활용이 제한됨
- 통계가 부정확함
- Random I/O 비용이 큼
- Index Lookup 후 Table Lookup이 너무 많이 발생함

따라서 성능 문제는 `Index 하나 추가하면 끝`이 아니다.

실행 계획을 봐야 한다.

---

## 11. Backend 실무 연결

사용자의 페이지네이션 Query를 예로 들면:

```sql
SELECT *
FROM orders
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;
```

`(user_id, created_at)` 같은 Composite Index가 Query 패턴과 잘 맞으면 특정 사용자의 최신 주문을 빠르게 찾는 데 도움이 된다.

반대로 Index 없이 대량 데이터를 Filter + Sort해야 한다면 응답 시간이 크게 늘 수 있다.

---

## 12. 60초 면접 답변

> B-Tree 계열 Index는 정렬된 Key를 여러 Page의 Tree 구조로 저장해서 Full Table Scan 없이 원하는 위치를 빠르게 찾게 해줍니다. 한 Node가 많은 Child를 가지므로 Tree 높이가 낮고 Equality뿐 아니라 Range Search에도 적합합니다. 다만 Index는 별도 Storage를 사용하고 INSERT, UPDATE, DELETE 때 함께 유지해야 하므로 공짜가 아닙니다. Composite Index는 Column 순서가 중요하고, Selectivity가 낮거나 결과 Row가 너무 많으면 Optimizer가 Index를 사용하지 않을 수도 있습니다. 그래서 실무에서는 Query 패턴과 실행 계획을 기준으로 Index를 설계해야 합니다.
