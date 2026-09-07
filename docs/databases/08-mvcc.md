# 08. MVCC — Multi-Version Concurrency Control

## 한 줄 정의

MVCC는 데이터를 한 순간의 단일 값으로만 관리하지 않고 **여러 Version을 유지해 Reader와 Writer가 서로 덜 막히도록 하는 동시성 제어 방식**입니다.

쉽게 말하면:

> 누군가 문서를 수정 중이어도 다른 사람은 자신이 볼 수 있는 이전 사본을 읽을 수 있게 한다.

---

## 왜 필요한가?

Lock만으로 동시성을 제어한다고 생각해 봅시다.

Writer가 Row를 수정하는 동안 Reader까지 모두 기다려야 한다면 읽기가 많은 Backend 시스템에서 병목이 커집니다.

MVCC는 Row의 Version을 관리해:

- Reader가 Writer를 덜 기다리게 하고
- Writer도 Reader 때문에 덜 막히게 하며
- Transaction마다 일관된 Snapshot을 제공할 수 있게 합니다.

단, MVCC가 Lock을 완전히 없애는 것은 아닙니다.

---

## 기본 아이디어

초기 Row:

```text
price = 1000
```

Transaction B가 다음과 같이 수정합니다.

```text
price = 1200
```

MVCC 시스템에서는 기존 Version을 즉시 덮어없애기보다 개념적으로 다음처럼 여러 Version이 존재할 수 있습니다.

```text
Version 1: price = 1000
Version 2: price = 1200
```

각 Transaction은 자신의 Snapshot과 가시성 규칙에 따라 어떤 Version을 볼지 결정합니다.

---

## Snapshot이란?

Snapshot은 단순히 전체 DB를 복사한 사진이 아닙니다.

보통은 Transaction ID / Version Metadata 등을 이용해:

> 이 Transaction이 볼 수 있는 Commit된 Version 집합

을 결정하는 논리적 기준입니다.

---

## Reader와 Writer

MVCC의 큰 장점은 흔히 다음 문장으로 요약합니다.

> Readers do not block writers, and writers do not block readers as much as traditional locking alone.

하지만 "절대 서로 안 막는다"고 말하면 과장입니다.

다음 경우에는 여전히 대기나 충돌이 발생할 수 있습니다.

- 같은 Row를 두 Writer가 수정
- `SELECT ... FOR UPDATE`
- DDL / Schema Lock
- Serializable 구현
- Index/Metadata 관련 내부 Lock

---

## Row Version은 어디에 저장되는가?

DBMS마다 다릅니다.

예를 들어 개념적으로:

- PostgreSQL: Tuple Version 자체를 Table에 유지하고 vacuum으로 정리
- MySQL/InnoDB: Undo Log를 이용해 이전 Version을 구성

따라서 MVCC를 설명할 때 특정 구현을 전체 DBMS의 공통 구조처럼 말하면 안 됩니다.

---

## Visibility

모든 Transaction이 최신 Version을 보는 것은 아닙니다.

예를 들어 Transaction A가 Snapshot을 잡았을 때 price=1000이었다면, 이후 B가 1200으로 Commit해도 A의 Isolation/DBMS 규칙에 따라 계속 1000을 볼 수 있습니다.

이 때문에 Repeatable Read 계열에서 같은 Transaction 안의 읽기가 안정적으로 유지될 수 있습니다.

---

## MVCC와 Isolation Level

MVCC는 **구현 메커니즘**이고 Isolation Level은 **보장하고 싶은 동작 규칙**입니다.

둘은 같은 개념이 아닙니다.

예를 들어:

- Read Committed를 MVCC로 구현할 수 있음
- Repeatable Read를 MVCC Snapshot으로 구현할 수 있음
- Serializable도 Snapshot + 충돌 감지 같은 방식으로 구현할 수 있음

즉:

> MVCC = 수단
> Isolation Level = 외부에 보이는 보장

---

## 오래 열린 Transaction의 문제

MVCC는 이전 Version을 유지해야 합니다.

그런데 아주 오래 열린 Transaction이 옛 Snapshot을 계속 필요로 하면 DB가 이전 Version을 쉽게 정리하지 못할 수 있습니다.

결과적으로:

- Dead Tuple 증가
- Undo History 증가
- Vacuum / Cleanup 지연
- Storage 증가
- 성능 저하

등이 발생할 수 있습니다.

따라서 Backend에서 Long Transaction은 매우 위험합니다.

---

## MVCC와 Garbage Collection

오래된 Version은 언젠가 제거되어야 합니다.

그렇지 않으면 DB 크기가 계속 증가합니다.

DBMS는 자신만의 방식으로 더 이상 누구에게도 보일 필요 없는 Version을 정리합니다.

이것이 PostgreSQL VACUUM, InnoDB purge 같은 메커니즘과 연결됩니다.

---

## MVCC가 Lost Update를 자동으로 막아주는가?

항상 그렇지는 않습니다.

두 Transaction이 같은 Snapshot의 값을 읽고 Application에서 계산 후 각각 Update하면 Lost Update 문제가 생길 수 있습니다.

따라서 다음이 필요할 수 있습니다.

- Atomic UPDATE
- `SELECT ... FOR UPDATE`
- Version Column 기반 Optimistic Lock
- 적절한 Isolation / Serializable

---

## Optimistic Lock과의 연결

Application에서 다음처럼 Version을 둔다고 합시다.

```text
id = 1
stock = 10
version = 5
```

읽은 후 다음과 같이 Update합니다.

```sql
UPDATE products
SET stock = 9,
    version = 6
WHERE id = 1
  AND version = 5;
```

Affected Row가 0이면 누군가 먼저 수정했다는 뜻입니다.

이 방식은 MVCC와 별개로 Application-level 충돌 감지 전략으로 사용할 수 있습니다.

---

## Backend 사례

### 주문 상세 조회

Writer가 주문 상태를 갱신 중이어도 Reader가 일관된 이전 Version을 볼 수 있어 읽기 병목을 줄일 수 있습니다.

### Batch 작업

아주 긴 Transaction으로 수십만 Row를 처리하면 오래된 Version 정리를 막을 수 있습니다.

### API Transaction

외부 HTTP 호출을 DB Transaction 안에서 오래 기다리면 Snapshot/Lock/Connection을 오래 점유할 수 있습니다.

따라서:

> DB Transaction은 가능한 짧게 유지한다.

---

## 흔한 오해

### 오해 1. MVCC면 Lock이 필요 없다

아닙니다. Writer-Writer 충돌과 명시적 Lock 등은 여전히 존재합니다.

### 오해 2. MVCC = Repeatable Read

아닙니다. MVCC는 여러 Isolation Level을 구현하는 데 사용할 수 있는 메커니즘입니다.

### 오해 3. Snapshot은 DB 전체 복사본이다

아닙니다. 보통 Version 가시성을 결정하는 논리적 기준입니다.

### 오해 4. 오래된 Version은 공짜다

아닙니다. Storage, cleanup, vacuum/purge 비용이 발생합니다.

---

## 면접용 60초 답변

> MVCC는 Multi-Version Concurrency Control로, Row의 여러 Version을 유지해 Reader와 Writer의 충돌을 줄이는 방식입니다. Transaction은 자신의 Snapshot과 visibility rule에 따라 볼 수 있는 Version을 선택하기 때문에 Writer가 새 값을 만드는 동안 Reader는 이전에 commit된 Version을 읽을 수 있습니다. 다만 MVCC가 Lock을 없애는 것은 아니며 같은 Row를 동시에 수정하는 Writer끼리는 여전히 충돌할 수 있습니다. 또한 오래 열린 Transaction은 오래된 Version 정리를 막아 vacuum이나 purge 부담을 키울 수 있으므로 Backend에서는 Transaction을 짧게 유지해야 합니다. Isolation Level은 보장이고 MVCC는 그것을 구현하는 수단이라는 점도 구분해야 합니다.
