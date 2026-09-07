# 04. Query Execution and EXPLAIN

## 한 줄 정의

SQL은 선언형 언어이기 때문에 개발자는 `무엇을 원하는지`를 쓰고, DB Optimizer가 `어떻게 가져올지` 실행 계획을 선택한다. `EXPLAIN`은 그 계획을 읽는 도구다.

---

## 1. SQL은 절차를 직접 명령하지 않는다

```sql
SELECT u.name
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE o.created_at >= ?;
```

개발자는 보통 다음을 직접 지정하지 않는다.

- users부터 읽을지 orders부터 읽을지
- Index Scan을 할지 Full Scan을 할지
- Nested Loop / Hash Join / Merge Join 중 무엇을 쓸지

Optimizer가 통계와 비용 모델을 바탕으로 계획을 만든다.

---

## 2. Logical Plan vs Physical Plan

### Logical
무슨 관계 연산이 필요한지.

- Filter
- Join
- Projection
- Aggregate
- Sort

### Physical
그 연산을 실제로 어떤 방식으로 수행할지.

예:

- Seq Scan
- Index Scan
- Nested Loop Join
- Hash Join
- Merge Join

면접에서는 `SQL 문장 자체`와 `실행 방식`을 구분해야 한다.

---

## 3. Optimizer는 무엇을 보고 판단하나?

대표적으로:

- Table Row 수
- Column 분포
- Selectivity
- Index 유무
- Join Cardinality 추정
- 정렬 필요 여부
- I/O / CPU 비용

이때 통계가 오래되거나 데이터 분포가 치우쳐 있으면 잘못된 계획을 선택할 수 있다.

---

## 4. Scan / Seek

### Full / Sequential Scan
Table의 많은 Page를 순차적으로 읽는다.

나쁜 것만은 아니다.

전체 Row의 상당 부분을 읽어야 한다면 오히려 Index Random Lookup보다 빠를 수 있다.

### Index Scan / Seek
Index를 이용해 필요한 범위를 찾는다.

하지만 Index를 썼다고 무조건 빠른 것은 아니다.

---

## 5. Join Algorithm

### Nested Loop

```text
for each row in outer:
    find matching rows in inner
```

Outer Row가 적고 Inner에 적절한 Index가 있으면 매우 효율적일 수 있다.

### Hash Join

한쪽 입력으로 Hash Table을 만들고 다른 쪽 Row를 Probe한다.

큰 Equality Join에 유리한 경우가 많다.

메모리가 부족하면 Spill이 발생할 수 있다.

### Merge Join

두 입력이 Join Key 순서로 정렬돼 있을 때 순차적으로 비교한다.

정렬된 대량 데이터에 유리할 수 있다.

어떤 Join이 항상 최고인 것은 아니다.

---

## 6. Sort는 왜 비쌀 수 있나?

```sql
ORDER BY created_at DESC
```

이미 필요한 순서의 Index가 없다면 DB가 결과를 별도로 Sort해야 할 수 있다.

데이터가 메모리에 들어가지 않으면 Disk Spill이 발생해 훨씬 느려질 수 있다.

따라서 Query에서:

- WHERE
- JOIN
- ORDER BY
- GROUP BY

를 함께 보고 Index를 설계해야 한다.

---

## 7. Cardinality Estimation

Optimizer가 `이 조건은 10건 정도 나오겠지`라고 예상했는데 실제로 100만 건이 나온다면 잘못된 Join 방식이나 메모리 크기를 선택할 수 있다.

이 예상 Row 수를 Cardinality Estimate라고 볼 수 있다.

실행 계획에서 중요한 질문은:

```text
Estimated Rows vs Actual Rows
```

차이가 큰가?

이다.

---

## 8. EXPLAIN vs EXPLAIN ANALYZE

일반적인 개념으로:

- `EXPLAIN`: Optimizer가 선택한 계획과 예상 비용을 보여준다.
- `EXPLAIN ANALYZE`: 실제 Query를 실행하고 실제 Row/시간 같은 정보를 함께 보여주는 DBMS가 많다.

주의: `ANALYZE`가 실제 Query를 실행한다면 UPDATE/DELETE 같은 문장에서는 운영 환경에서 위험할 수 있다. DBMS 문서를 확인해야 한다.

---

## 9. 실행 계획에서 먼저 보는 것

실무에서는 다음 순서로 보면 좋다.

1. 예상보다 많은 Row를 읽고 있는가?
2. Full Scan이 의도된 것인가?
3. Index 조건이 제대로 적용되는가?
4. Join 순서와 Join Algorithm이 적절한가?
5. 반복되는 Key Lookup이 많은가?
6. Sort / Hash가 Disk로 Spill되는가?
7. Estimated Rows와 Actual Rows가 크게 다른가?

---

## 10. SARGable 조건

Index가 있어도 조건 표현 방식 때문에 효율적인 탐색이 어려울 수 있다.

예:

```sql
WHERE YEAR(created_at) = 2026
```

Column에 함수를 적용하면 DBMS에 따라 일반 Index를 바로 활용하기 어려울 수 있다.

범위 조건으로 바꾸면 더 유리할 수 있다.

```sql
WHERE created_at >= '2026-01-01'
  AND created_at <  '2027-01-01'
```

이처럼 Search Argument로 Index Range를 잡기 쉬운 조건을 흔히 SARGable하다고 부른다.

---

## 11. Pagination과 실행 계획

```sql
ORDER BY id
LIMIT 20 OFFSET 1000000
```

큰 OFFSET은 앞의 많은 Row를 건너뛰기 위해 큰 작업이 필요할 수 있다.

Keyset Pagination:

```sql
WHERE id > :last_id
ORDER BY id
LIMIT 20
```

은 적절한 Index가 있으면 이전 위치 다음부터 직접 읽을 수 있어 대규모 페이지네이션에 더 유리할 수 있다.

---

## 12. ORM에서도 실행 계획이 중요한 이유

Entity Framework나 다른 ORM을 쓰더라도 DB가 실행하는 것은 결국 SQL이다.

문제 예:

- N+1 Query
- 필요 없는 Column 전체 조회
- Client-side evaluation
- 잘못된 Include / Join
- Index와 맞지 않는 Filter

따라서 Backend Engineer는 ORM 코드뿐 아니라 생성 SQL과 실행 계획도 볼 수 있어야 한다.

---

## 13. 60초 면접 답변

> SQL은 선언형 언어이기 때문에 개발자는 원하는 결과를 표현하고 Optimizer가 통계와 비용 모델을 기반으로 Scan 방식, Join 순서와 Join Algorithm 같은 Physical Plan을 결정합니다. EXPLAIN을 볼 때는 Index 사용 여부 하나만 보는 것이 아니라 실제로 몇 Row를 읽는지, Key Lookup이 반복되는지, Sort나 Hash가 Spill되는지, Estimated Rows와 Actual Rows가 크게 다른지 확인해야 합니다. Full Scan도 많은 Row가 필요하면 합리적일 수 있습니다. 실무에서는 느린 Query를 발견하면 생성 SQL과 실행 계획을 같이 보고 병목 원인을 확인합니다.
