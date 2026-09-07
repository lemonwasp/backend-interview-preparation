# 01. Relational Model and Keys

## 한 줄 정의

관계형 데이터베이스는 데이터를 `Table`이라는 관계 형태로 저장하고, `Key`를 사용해 Row를 식별하거나 Table 사이의 관계를 연결한다.

---

## 1. 왜 관계형 모델이 필요한가?

예를 들어 쇼핑몰에 다음 정보가 있다고 하자.

- 사용자
- 주문
- 상품
- 결제

이 데이터를 한 파일에 전부 섞어 넣으면 중복이 많아지고 수정할 때 일관성을 유지하기 어렵다.

관계형 모델은 데이터를 의미 있는 단위로 나누고, 각 Table을 Key로 연결한다.

```text
users
id | name

orders
id | user_id | total_price
```

여기서 `orders.user_id`가 `users.id`를 가리키면 주문이 어떤 사용자에게 속하는지 표현할 수 있다.

---

## 2. Table / Row / Column

### Table
같은 종류의 데이터를 모아 둔 구조다.

### Row
하나의 실제 Entity 또는 Record다.

### Column
Entity가 가진 속성이다.

예:

```text
users
------------------------------------------------
id | email             | name
1  | a@example.com     | Alice
2  | b@example.com     | Bob
```

- Table: `users`
- Row: Alice 한 명의 사용자 데이터
- Column: `id`, `email`, `name`

---

## 3. Primary Key

Primary Key는 **Row 하나를 유일하게 식별하는 Key**다.

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) NOT NULL
);
```

### 좋은 Primary Key가 가져야 할 성질

- 유일해야 한다.
- NULL이 될 수 없다.
- 가능한 한 안정적으로 변하지 않아야 한다.

### Natural Key vs Surrogate Key

#### Natural Key
업무 데이터 자체를 Key로 사용하는 방식.

예:

- 이메일
- 주민등록번호
- 상품 코드

문제는 업무 규칙이 변하면 Key도 변할 수 있다는 점이다.

#### Surrogate Key
업무 의미가 없는 별도 식별자를 만든다.

예:

```text
id = 92831
```

Backend에서는 `BIGINT`, UUID 같은 Surrogate Key를 자주 사용한다.

---

## 4. Foreign Key

Foreign Key는 다른 Table의 Key를 참조해 관계를 표현한다.

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

이 제약이 있으면 존재하지 않는 `user_id`를 주문에 넣는 것을 DB가 막을 수 있다.

이 성질을 **Referential Integrity**라고 한다.

---

## 5. Candidate Key / Alternate Key

Row를 유일하게 식별할 수 있는 후보가 여러 개일 수 있다.

예를 들어 users Table에서:

- `id`
- `email`

둘 다 유일하다면 둘 모두 Candidate Key가 될 수 있다.

그중 하나를 Primary Key로 선택하고 나머지는 Alternate Key로 볼 수 있다.

실무에서는 보통 다음처럼 표현한다.

```sql
id BIGINT PRIMARY KEY,
email VARCHAR(255) UNIQUE NOT NULL
```

---

## 6. Composite Key

두 개 이상의 Column을 합쳐야 Row를 유일하게 식별할 수 있는 경우다.

예:

```text
order_items
order_id | product_id | quantity
```

한 주문에 같은 상품이 한 줄만 존재하도록 설계한다면:

```sql
PRIMARY KEY (order_id, product_id)
```

이렇게 둘을 합쳐 Key로 사용할 수 있다.

---

## 7. UNIQUE Constraint와 Primary Key 차이

둘 다 중복을 막는 데 사용할 수 있지만 의미가 다르다.

- Primary Key: 해당 Row의 대표 식별자
- UNIQUE: 특정 Column 조합이 중복되지 않도록 보장하는 제약

Table에는 일반적으로 Primary Key 하나가 있고 UNIQUE Constraint는 여러 개 둘 수 있다.

NULL 처리 방식은 DBMS에 따라 차이가 있을 수 있으므로 특정 구현을 확인해야 한다.

---

## 8. 관계의 종류

### 1:N
한 사용자가 여러 주문을 가진다.

```text
users 1 ---- N orders
```

### 1:1
사용자와 사용자 상세 정보처럼 하나씩 연결할 수 있다.

### N:M
학생과 강의처럼 양쪽 모두 여러 개와 관계를 가질 수 있다.

관계형 DB에서는 중간 Table을 둔다.

```text
students
courses
student_courses
```

---

## 9. Backend에서 왜 중요할까?

잘못된 관계 모델은 Application 코드가 아무리 좋아도 문제를 만든다.

예:

- 중복 사용자 생성
- 존재하지 않는 사용자의 주문
- 관계 삭제 후 고아 데이터
- Join하기 어려운 구조
- 변경 시 여러 위치를 동시에 수정해야 하는 데이터 중복

DB Schema는 단순 저장 형식이 아니라 **데이터가 가질 수 있는 상태의 규칙**이다.

---

## 10. Foreign Key를 항상 사용해야 하나?

장점:

- Referential Integrity를 DB가 강제한다.
- 잘못된 데이터가 들어가는 것을 막는다.

비용/고려사항:

- 쓰기 시 제약 검사 비용이 있다.
- 대규모 분산 시스템이나 Sharding 구조에서는 다른 DB/Shard 간 FK를 직접 사용하기 어렵다.
- 삭제 순서와 Cascade 정책을 설계해야 한다.

따라서 `Foreign Key가 무조건 좋다/나쁘다`가 아니라 시스템 경계와 운영 요구를 보고 판단해야 한다.

---

## 11. 면접에서 자주 나오는 함정

### "Primary Key = Index인가요?"

개념적으로는 다르다.

- Primary Key는 데이터 무결성을 위한 Key Constraint다.
- Index는 검색을 빠르게 하기 위한 자료구조다.

다만 실제 DBMS는 Primary Key를 지원하기 위해 Index를 자동 생성하는 경우가 많다.

### "UUID와 BIGINT 중 뭐가 더 좋나요?"

상황에 따라 다르다.

- BIGINT: 작고 순차적이라 Index locality 측면에서 유리한 경우가 많다.
- UUID: 분산 환경에서 중앙 ID 발급 없이 생성하기 쉽다.

UUID도 버전과 생성 방식에 따라 순서 특성이 다르므로 단순히 `UUID는 항상 느리다`고 말하면 안 된다.

---

## 12. 60초 면접 답변

> 관계형 데이터베이스는 데이터를 Table, Row, Column 구조로 표현하고 Key를 통해 Row 식별과 Table 간 관계를 관리합니다. Primary Key는 Row의 대표 식별자이고 Foreign Key는 다른 Table의 Key를 참조해 Referential Integrity를 보장합니다. Candidate Key 중 하나를 Primary Key로 선택할 수 있고, 여러 Column을 합친 Composite Key도 사용할 수 있습니다. Backend에서는 Key 설계가 중복 방지, 데이터 일관성, Join 구조와 직접 연결되기 때문에 단순한 Schema 문법보다 데이터 무결성 규칙으로 이해하는 것이 중요합니다.
