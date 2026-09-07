# 17. ORM / N+1

## 한 줄 핵심

ORM은 객체 모델과 관계형 DB 사이의 반복적인 매핑 작업을 줄여주지만, SQL 실행을 없애는 도구가 아니므로 실제 생성되는 Query 수와 실행 계획을 이해해야 한다.

## ORM이 해주는 일

ORM은 대략 다음을 자동화한다.

- Row ↔ Object Mapping
- Change Tracking
- Query Builder
- Relationship Mapping
- Transaction 연동

예: C# Entity Framework, Java Hibernate/JPA.

## N+1 문제

게시글 100개와 작성자를 보여준다고 하자.

먼저 게시글 조회:

```sql
SELECT * FROM posts LIMIT 100;
```

그 다음 각 게시글마다 작성자 조회:

```sql
SELECT * FROM users WHERE id = ?;
```

이렇게 되면:

```text
1개 Posts Query
+ 100개 User Query
= 101 Queries
```

이것이 전형적인 N+1이다.

## 왜 느린가

Query 하나가 빠르더라도:

- DB round trip 반복
- Connection 사용 시간 증가
- Query parsing/execution 반복
- DB와 App CPU 증가

때문에 전체 latency가 커질 수 있다.

## Lazy Loading이 항상 나쁜가?

아니다.

Lazy Loading은 실제로 필요할 때만 연관 데이터를 가져오는 유용한 전략이다. 문제는 반복문 안에서 의도치 않게 많은 Query를 발생시키는 경우다.

## 해결 방법

### 1. Eager Loading / JOIN

```sql
SELECT p.*, u.*
FROM posts p
JOIN users u ON p.user_id = u.id;
```

ORM에서는 Include / Fetch Join 같은 기능을 사용할 수 있다.

### 2. Batch Loading

```sql
SELECT * FROM users WHERE id IN (...);
```

N개의 Query를 몇 개의 Batch Query로 줄인다.

### 3. Projection

화면에 필요한 Column만 직접 조회한다.

```sql
SELECT p.id, p.title, u.name
FROM posts p
JOIN users u ON ...;
```

## Eager Loading도 공짜는 아니다

관계를 여러 개 JOIN하면 Row 수가 폭발하는 Cartesian Explosion이 생길 수 있다.

예:

```text
Order 1개
× Items 20개
× Tags 10개
= 200 Rows
```

따라서 `무조건 JOIN`이 정답도 아니다.

## ORM 성능 문제에서 볼 것

- 실제 SQL
- Query 개수
- DB round trip
- 읽는 Row/Column 수
- Index 사용 여부
- Tracking 필요 여부
- Pagination 적용 여부

## Tracking vs No Tracking

변경할 필요 없는 조회에서 Change Tracking을 끄면 ORM overhead를 줄일 수 있다. 다만 구체적인 API와 효과는 ORM마다 다르므로 면접에서는 원리 중심으로 설명한다.

## N+1을 발견하는 법

- SQL Logging
- APM Trace
- 동일 패턴 Query 반복
- Request당 Query Count

## Backend 예시

`GET /orders` 하나가 30ms였는데 데이터가 늘면서 800ms가 되었다.

APM을 보니:

```text
SELECT orders ... 1회
SELECT customer ... 100회
SELECT items ... 100회
```

이 경우 CPU 최적화 전에 Query Pattern부터 고쳐야 한다.

## 60초 면접 답변

ORM은 객체와 관계형 DB 사이의 Mapping과 Query 생성을 편하게 해주지만 실제 SQL 비용을 없애지는 않습니다. 대표적인 문제는 N+1으로, 부모 N개를 조회한 뒤 각 Row의 연관 데이터를 개별 조회해서 총 N+1개의 Query가 발생하는 패턴입니다. 해결 방법으로는 Fetch Join/Eager Loading, Batch Loading, Projection 등이 있습니다. 다만 모든 관계를 한 번에 JOIN하면 Cartesian Explosion이 생길 수 있어서 실제 SQL, Query Count, Row 수와 실행 계획을 측정해 선택해야 합니다. ORM을 쓸수록 오히려 생성 SQL을 볼 수 있어야 한다고 생각합니다.
