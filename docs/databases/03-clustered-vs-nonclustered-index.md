# 03. Clustered vs Non-clustered Index

## 한 줄 정의

Clustered Index는 Table의 실제 Row 저장 순서 또는 Row가 저장된 주된 구조와 강하게 결합된 Index이고, Non-clustered Index는 별도의 Index 구조에서 Row 위치 또는 Row를 찾을 수 있는 Key를 가리키는 Index다.

> 정확한 구현은 DBMS마다 다르므로 개념과 제품별 구현을 구분해서 이해해야 한다.

---

## 1. 왜 이 구분이 필요한가?

Index가 Key를 찾았다고 끝이 아닐 수 있다.

```sql
SELECT name, address
FROM users
WHERE email = ?;
```

`email` Index에서 사용자를 찾은 뒤 실제 `name`, `address`가 있는 Row를 다시 읽어야 할 수 있다.

즉 두 단계가 생긴다.

```text
Secondary Index
    ↓
Row locator / Primary key
    ↓
Actual row
```

이 추가 접근이 Query 비용에 큰 영향을 줄 수 있다.

---

## 2. Clustered Index의 핵심 감각

개념적으로는 `Table 데이터 자체가 Index의 Leaf와 함께 정렬된 형태`라고 생각하면 쉽다.

```text
Clustered B+Tree
Root
 ↓
Intermediate
 ↓
Leaf = actual row data
```

따라서 Clustered Key로 찾았을 때 별도의 Table lookup 없이 Row에 도달할 수 있다.

### 중요한 특징

- Table의 실제 Row 저장 구조와 강하게 연결된다.
- 하나의 Table에 여러 개의 물리적 정렬을 동시에 가질 수 없으므로 보통 Clustered 구조는 하나뿐이다.
- Range Scan에 유리할 수 있다.

---

## 3. Non-clustered Index

Non-clustered Index는 Table 데이터와 별도의 구조다.

```text
Non-clustered Index
email -> row locator
```

Leaf에는 보통:

- Indexed Key
- Row locator 또는 Primary Key
- Included Column 일부

등이 들어갈 수 있다.

그 뒤 필요한 Row 전체를 읽기 위해 다른 구조로 다시 이동할 수 있다.

---

## 4. Key Lookup / Bookmark Lookup

예를 들어:

```sql
SELECT *
FROM users
WHERE email = ?;
```

email Non-clustered Index가 있다면:

1. email Index에서 Key를 찾는다.
2. Row locator를 얻는다.
3. 실제 Row를 다시 읽는다.

이 두 번째 단계가 Key Lookup 또는 Bookmark Lookup 같은 이름으로 표현될 수 있다.

조회 결과가 몇 건이면 괜찮지만 수십만 건이면 반복 Lookup 비용이 커질 수 있다.

---

## 5. Covering Index와의 연결

Query에 필요한 Column이 Index 안에 모두 있으면 실제 Row를 다시 읽을 필요가 없다.

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(name);
```

```sql
SELECT name
FROM users
WHERE email = ?;
```

이 경우 DBMS에 따라 Index만 읽고 결과를 만들 수 있다.

즉 Covering Index는 Non-clustered Index의 추가 Row Lookup 비용을 줄이는 전략이 될 수 있다.

---

## 6. Primary Key와 Clustered Index는 같은가?

항상 같은 개념은 아니다.

- Primary Key: Constraint
- Clustered Index: Storage / Access Structure

일부 DBMS는 Primary Key를 기본적으로 Clustered Index로 만들 수 있지만, 제품과 설정에 따라 다르다.

### MySQL InnoDB

InnoDB는 Primary Key를 Clustered Index로 사용한다.

Secondary Index Leaf에는 Primary Key 값이 들어간다.

따라서 Secondary Index 조회는 개념적으로:

```text
Secondary Index
   ↓ PK
Primary/Clustered Index
   ↓
Row
```

### SQL Server

Clustered Index를 명시적으로 선택할 수 있고 Primary Key와 Clustered Index는 별개 개념이다.

따라서 면접에서 특정 DB를 말하지 않고 `Primary Key는 무조건 Clustered다`라고 답하면 위험하다.

---

## 7. Clustered Key를 선택할 때 중요한 점

Clustered Key가 다른 Index에서 Row locator 역할을 한다면 Key 크기가 전체 Index 크기에 영향을 줄 수 있다.

좋은 Clustered Key는 일반적으로:

- 작고
- 안정적이고
- 가능한 한 변경이 적고
- 삽입 패턴이 지나치게 Random하지 않은 것

이 유리할 수 있다.

하지만 시스템 요구와 DBMS 특성을 고려해야 한다.

---

## 8. Random UUID가 왜 화제가 되나?

완전히 Random한 UUID를 Clustered Key로 사용하면 삽입 위치가 Tree 전체에 흩어질 수 있다.

가능한 문제:

- Page Split 증가
- Cache locality 저하
- Fragmentation 증가
- 더 큰 Key 크기

하지만 이것도 `UUID는 무조건 나쁘다`는 뜻이 아니다.

- Sequential UUID
- UUIDv7
- 별도 Surrogate Key

같은 선택지가 있고 workload에 따라 판단해야 한다.

---

## 9. Range Query

예:

```sql
SELECT *
FROM orders
WHERE created_at BETWEEN ? AND ?;
```

`created_at`이 Clustered 순서와 잘 맞으면 인접 Page를 순차적으로 읽을 수 있어 Range Scan에 유리할 수 있다.

반대로 Non-clustered Index로 많은 Row를 찾은 후 각 Row마다 Random Lookup을 해야 한다면 비용이 커질 수 있다.

---

## 10. Backend에서 보는 실제 질문

### 질문

> Index가 있는데 왜 Query가 느릴까요?

가능한 답:

- Index Scan 후 수많은 Key Lookup 발생
- Selectivity가 낮음
- 반환 Row가 너무 많음
- Covering되지 않음
- Clustered Key가 너무 큼
- Random I/O가 많음

즉 `Index를 탔다`는 사실만으로 충분하지 않다.

---

## 11. 60초 면접 답변

> Clustered Index는 Table의 실제 Row 저장 구조와 강하게 연결되어 있고 Leaf에서 Row 자체에 도달할 수 있는 구조라고 이해하면 됩니다. 반면 Non-clustered Index는 별도의 구조라서 Indexed Key를 찾은 뒤 Row locator나 Primary Key를 통해 실제 Row를 다시 읽는 과정이 필요할 수 있습니다. 이 추가 접근이 Key Lookup 비용입니다. Query에 필요한 Column을 Index가 모두 포함하면 Covering Index로 Lookup을 줄일 수 있습니다. 다만 Primary Key와 Clustered Index는 같은 개념이 아니며 DBMS마다 구현이 다르기 때문에 제품별 차이를 구분해서 설명해야 합니다.
