# 11. Normalization

## 한 줄 설명

Normalization은 **같은 사실을 여러 곳에 중복 저장해서 생기는 수정 불일치를 줄이기 위해 데이터를 관계와 의존성에 따라 분리하는 설계 원칙**입니다.

쉽게 말하면:

> 같은 정보가 여러 칸에 복붙되어 있다면, 언젠가 한 군데만 수정되고 다른 곳은 안 바뀔 가능성이 생긴다.

그래서 관계형 DB에서는 중복을 줄이고 한 사실의 원천을 명확히 만들려고 합니다.

---

## 왜 필요한가

다음과 같은 Table을 생각해봅시다.

```text
orders
- order_id
- customer_id
- customer_name
- customer_address
- product_id
- product_name
- product_price
- quantity
```

한 고객이 주문을 100번 하면 이름과 주소가 100번 반복될 수 있습니다.

주소가 바뀌면 100개 Row를 전부 고쳐야 합니다.

여기서 일부만 수정되면 서로 다른 주소가 공존합니다.

이것이 대표적인 **Update Anomaly**입니다.

---

## 대표적인 이상 현상

### 1. Update Anomaly

같은 사실이 여러 Row에 중복되어 있어 여러 곳을 함께 수정해야 합니다.

### 2. Insert Anomaly

아직 주문이 없는 고객 정보만 저장하고 싶은데 주문 Table 구조상 저장하기 어려울 수 있습니다.

### 3. Delete Anomaly

마지막 주문 Row를 삭제했더니 고객 정보까지 같이 사라질 수 있습니다.

---

## Functional Dependency

Normalization의 핵심은 이름 외우기가 아니라 **무엇이 무엇을 결정하는가**를 보는 것입니다.

예:

```text
customer_id -> customer_name, customer_address
product_id  -> product_name, product_price
order_id    -> customer_id, ordered_at
```

`customer_id`를 알면 해당 고객의 이름과 주소가 결정됩니다.

이런 관계를 Functional Dependency라고 합니다.

---

## 1NF

### 핵심

한 Column에는 하나의 값이 들어가도록 합니다.

나쁜 예:

```text
phone_numbers = "010-..., 02-..., 031-..."
```

이런 값을 하나의 문자열에 몰아넣으면 검색과 제약 조건 적용이 어려워집니다.

주의:

1NF를 단순히 "무조건 Array를 쓰면 안 된다"로 외우면 안 됩니다. 현대 DB는 JSON/Array 타입도 제공하므로 실제 설계에서는 조회 패턴과 무결성 요구를 함께 판단해야 합니다.

---

## 2NF

### 핵심

Composite Key를 사용하는 Table에서 일반 Column이 **Key의 일부에만 의존하는 Partial Dependency**를 제거합니다.

예:

```text
order_items
PK = (order_id, product_id)

product_name -> 실제로는 product_id에만 의존
```

`product_name`은 `(order_id, product_id)` 전체가 아니라 `product_id`에만 의존합니다.

따라서 Product Table로 분리할 수 있습니다.

---

## 3NF

### 핵심

Key가 아닌 Column이 다른 Key가 아닌 Column을 통해 간접적으로 결정되는 **Transitive Dependency**를 줄입니다.

예:

```text
employees
- employee_id
- department_id
- department_name
```

```text
employee_id -> department_id
department_id -> department_name
```

`department_name`은 Employee 자체보다 Department에 속한 사실입니다.

따라서 Department Table로 분리하는 편이 자연스럽습니다.

---

## BCNF

BCNF는 3NF보다 더 엄격하게 Functional Dependency의 determinant가 Candidate Key인지 확인합니다.

기술면접에서 핵심은 수학적 정의를 암송하는 것보다 다음을 말할 수 있는 것입니다.

> Normalization은 중복과 이상 현상을 줄이고 데이터 무결성을 개선하지만, Join 수 증가와 조회 복잡도라는 비용이 있다.

---

## Normalization 이후 구조

```text
customers
- customer_id PK
- name
- address

products
- product_id PK
- name
- price

orders
- order_id PK
- customer_id FK
- ordered_at

order_items
- order_id FK
- product_id FK
- quantity
- price_at_purchase
```

여기서 중요한 점:

`price_at_purchase`는 일부러 Product의 현재 가격을 복사할 수 있습니다.

왜냐하면 주문 당시 가격은 **현재 상품 가격과 다른 역사적 사실**이기 때문입니다.

즉 중복처럼 보여도 비즈니스 의미가 다르면 무조건 제거하면 안 됩니다.

---

## Denormalization

Denormalization은 성능이나 조회 단순화를 위해 일부 데이터를 의도적으로 중복 저장하는 것입니다.

예:

- 통계용 집계값
- 검색 화면에 자주 필요한 표시명
- Read Model
- Data Warehouse
- 캐시성 Summary Table

하지만 대가가 있습니다.

```text
Read 성능 개선
        ↕
Write 복잡성 / 동기화 비용 / 불일치 위험
```

---

## Backend에서 언제 Normalization이 유리한가

특히 OLTP 시스템에서:

- 주문
- 결제
- 계정
- 재고
- 권한

처럼 정확한 정합성이 중요한 데이터는 일반적으로 정규화가 좋은 출발점입니다.

반대로 읽기 전용 분석이나 매우 빈번한 조회에서는 의도적 Denormalization이 합리적일 수 있습니다.

---

## 흔한 오해

### 오해 1. Normalize할수록 무조건 좋은 설계다

아닙니다.

정규화는 무결성과 중복을 개선하지만 Join과 Query 복잡도가 증가합니다.

### 오해 2. Denormalization은 나쁜 설계다

아닙니다.

의도적이고 동기화 전략이 명확하다면 성능을 위한 합리적 선택일 수 있습니다.

### 오해 3. 중복 Column은 항상 제거해야 한다

아닙니다.

주문 당시 가격처럼 의미적으로 다른 Snapshot 데이터는 유지해야 할 수 있습니다.

---

## 면접 60초 답변

> Normalization은 데이터 중복으로 인해 발생하는 Insert, Update, Delete anomaly를 줄이기 위해 Functional Dependency에 따라 Table을 분리하는 설계 원칙입니다. 1NF, 2NF, 3NF로 갈수록 중복과 비정상적인 Dependency를 줄여 데이터 무결성을 높일 수 있습니다. 반면 Table이 늘고 Join 비용과 Query 복잡도가 커질 수 있습니다. 그래서 OLTP의 핵심 데이터는 정규화된 구조를 기본으로 하고, 조회 성능이 중요한 부분은 Cache, Read Model, Summary Table 같은 방식으로 의도적으로 Denormalization할 수 있습니다. 중요한 것은 정규화 단계 자체를 외우는 것보다 어떤 사실의 원천이 어디인지, 중복이 어떤 불일치를 만들 수 있는지 설명하는 것입니다.
