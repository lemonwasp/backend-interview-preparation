# 16. DB Connection Pool / Transaction Boundary

## 한 줄 핵심

DB Connection Pool은 비싼 연결을 재사용하는 장치이면서 동시에 DB로 들어가는 동시 요청 수를 제한하는 보호 장치이고, Transaction Boundary는 어떤 작업들을 하나의 원자적 단위로 묶을지 정하는 설계 결정이다.

## 왜 Connection Pool이 필요한가

매 Query마다 새 DB Connection을 만들면 TCP/TLS/인증/세션 초기화 비용이 반복된다. Pool은 미리 열어 둔 Connection을 빌리고 반환해 이 비용을 줄인다.

하지만 Pool은 무한하지 않다.

```text
Request
  ↓
Pool에서 Connection 획득
  ↓
Query / Transaction
  ↓
Connection 반환
```

Pool이 다 차면 새 Request는 대기하거나 timeout된다.

## Pool Exhaustion

대표 원인:

- Connection Leak
- 너무 긴 Transaction
- 느린 Query
- Lock Wait / Deadlock
- 외부 API를 기다리면서 DB Connection을 잡고 있음
- Application Instance 수 증가에 비해 DB capacity가 작음

예를 들어 App Instance가 20대이고 각 Pool Size가 50이면 이론상 DB에 최대 1000 Connection이 몰릴 수 있다.

그래서 `Pool Size는 Application 하나만 보고 정하면 안 된다.`

## Transaction Boundary가 중요한 이유

좋지 않은 예:

```text
BEGIN
UPDATE orders ...
외부 결제 API 호출 3초 대기
UPDATE payments ...
COMMIT
```

외부 API가 느려질수록 Transaction이 길어지고:

- Lock 보유 시간이 길어짐
- MVCC 오래된 Version 정리 지연
- Connection 반환 지연
- Pool Exhaustion
- Deadlock 가능성 증가

따라서 일반적으로 Transaction 안에는 꼭 필요한 DB 작업만 넣고 외부 Network I/O를 무심코 포함하지 않는다.

## Transaction은 Request와 같은 범위인가?

항상 그렇지 않다.

하나의 HTTP Request 안에서도 Transaction이 여러 개일 수 있고, 반대로 하나의 비즈니스 Workflow가 여러 Request/Service에 걸칠 수 있다.

중요한 질문은:

> 어떤 상태 변화가 함께 성공하거나 함께 실패해야 하는가?

이다.

## ORM과 Connection Scope

ORM의 `DbContext`, `Session`, `EntityManager` 같은 객체는 논리적 작업 단위를 표현할 수 있지만, 그것 자체가 항상 하나의 물리 Connection을 전체 수명 동안 점유한다는 뜻은 아니다. 실제 동작은 ORM/Provider/Transaction 사용 방식에 따라 달라진다.

따라서 면접에서는 프레임워크 세부 구현을 단정하기보다 다음을 말하는 편이 안전하다.

> Transaction이 열려 있는 동안에는 Connection과 Lock 같은 DB resource 점유 시간이 길어질 수 있으므로 Scope를 짧게 유지해야 한다.

## Pool Size를 크게 하면 해결되는가?

아니다.

Pool Size를 무작정 늘리면:

- DB CPU/Memory 사용량 증가
- Lock Contention 증가
- Query Scheduler 경쟁 증가
- 장애 시 더 많은 요청이 동시에 DB로 진입

할 수 있다.

Pool은 Queue와 마찬가지로 capacity control이다.

## 관찰해야 할 지표

- Active / Idle Connection 수
- Pool Waiter 수
- Connection acquisition latency
- Query latency
- Transaction duration
- Lock wait time
- DB CPU / Memory

## 장애 시나리오

평소 Query는 20ms인데 갑자기 한 Query가 Lock 때문에 5초씩 기다린다.

그 결과:

```text
Lock Wait 증가
→ Transaction 길어짐
→ Connection 반환 지연
→ Pool 고갈
→ 다른 정상 Request도 Connection 획득 대기
→ 전체 API latency 증가
```

즉 DB Pool Exhaustion은 원인이 아니라 2차 증상일 수 있다.

## 60초 면접 답변

DB Connection Pool은 Connection 생성 비용을 줄이기 위해 연결을 재사용하는 구조이면서 DB로 들어가는 동시성을 제한하는 보호 장치입니다. Pool이 너무 작으면 acquisition wait가 커지고, 너무 크면 DB contention을 키울 수 있기 때문에 전체 Application Instance 수와 DB capacity를 함께 봐야 합니다. Transaction Boundary도 중요합니다. Transaction이 길면 Lock, MVCC version, Connection을 오래 점유해서 Pool Exhaustion과 Deadlock으로 이어질 수 있습니다. 특히 외부 API 호출처럼 느리고 불확실한 Network I/O를 DB Transaction 안에 오래 포함하지 않고, 함께 원자적으로 처리해야 하는 DB 작업만 짧게 묶는 것이 기본 원칙입니다.
