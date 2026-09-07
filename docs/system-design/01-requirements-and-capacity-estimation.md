# 01. Requirements and Capacity Estimation

System Design에서 가장 먼저 해야 할 일은 기술을 고르는 것이 아니라 **문제를 숫자와 요구사항으로 바꾸는 것**이다.

## 1. 가장 쉬운 비유

식당을 설계한다고 생각해보자.

- 하루 손님이 100명인지 10만 명인지
- 점심시간에 몰리는지
- 주문 결과가 1초 안에 나와야 하는지
- 주문 기록을 절대 잃으면 안 되는지

이걸 모른 채 주방 크기나 직원 수를 정하면 설계가 맞을 수 없다.

Backend System Design도 같다.

## 2. Functional vs Non-functional Requirements

### Functional Requirements
서비스가 **무엇을 해야 하는가**.

예:
- 사용자가 글을 작성한다.
- 피드를 조회한다.
- 이미지를 업로드한다.
- 알림을 보낸다.

### Non-functional Requirements
서비스가 **어떤 품질로 동작해야 하는가**.

예:
- p99 latency 300ms 이하
- 월 가용성 99.9%
- 강한 일관성이 필요한 결제
- eventual consistency를 허용하는 조회수
- 특정 지역 장애에도 서비스 지속

## 3. Capacity Estimation의 목적

정확한 예언이 아니라 **병목의 크기와 설계 방향을 판단하는 것**이다.

대표적으로 계산하는 값:
- DAU / MAU
- requests per second (RPS/QPS)
- read/write ratio
- payload size
- storage growth
- bandwidth
- peak traffic

## 4. 평균과 Peak를 구분하라

하루 요청이 86,400,000건이라면 평균은 약 1,000 RPS다.

하지만 실제 트래픽은 일정하지 않다.

예:
- 평균 1,000 RPS
- 평소 peak 3,000 RPS
- 이벤트 순간 10,000 RPS

따라서 `average RPS`만 보고 서버 수를 정하면 장애가 날 수 있다.

## 5. 간단한 계산 예시

가정:
- DAU 1,000,000명
- 사용자당 하루 평균 API 요청 20회
- 총 요청 = 20,000,000/day

평균 RPS:

```text
20,000,000 / 86,400 ≈ 231 RPS
```

Peak가 평균의 5배라면:

```text
≈ 1,155 RPS
```

읽기:쓰기 = 9:1이라면 peak에서 대략:

```text
Read ≈ 1,040 RPS
Write ≈ 115 RPS
```

이 정도만 알아도 cache/read replica 필요성 판단이 쉬워진다.

## 6. Storage Estimation

예:
- 하루 신규 이미지 100,000개
- 평균 이미지 2MB

```text
100,000 × 2MB = 200GB/day
```

1년:

```text
≈ 73TB/year
```

여기에 replication, backup, metadata를 더하면 실제 필요 용량은 더 커진다.

## 7. Bandwidth Estimation

예:
- peak 1,000 responses/sec
- response 평균 100KB

```text
1,000 × 100KB = 100MB/s
```

대략 800Mbps 수준이다.

그래서 큰 정적 파일은 application server보다 Object Storage + CDN으로 빼는 것이 자연스럽다.

## 8. Latency Budget

전체 API 목표가 300ms라면 downstream 각각에 300ms를 주면 안 된다.

예:

```text
Client → API Gateway    20ms
App processing          40ms
DB                      80ms
Cache                   10ms
External API            80ms
Network + margin        70ms
-----------------------------
Total                  300ms
```

이 관점은 Networking에서 배운 Timeout Budget과 연결된다.

## 9. Availability를 숫자로 이해하기

99.9% availability라면 한 달 약 43분 정도의 downtime을 허용한다.

99.99%라면 약 4분대다.

숫자가 높아질수록:
- redundancy
- failover
- multi-AZ
- monitoring
- operational complexity

비용이 급격히 증가한다.

## 10. Consistency Requirement도 요구사항이다

모든 데이터를 Strong Consistency로 만들 필요는 없다.

예:
- 계좌 잔액: 강한 정확성 중요
- 좋아요 수: 약간 늦어도 됨
- 검색 색인: eventual consistency 허용 가능

정합성 수준이 architecture 비용을 결정한다.

## 11. 면접에서 좋은 진행 순서

1. 핵심 기능을 확인한다.
2. 규모를 확인한다.
3. Read/Write 비율을 확인한다.
4. Latency / Availability / Consistency를 확인한다.
5. 데이터 크기와 증가량을 추정한다.
6. 가장 먼저 병목이 될 부분을 찾는다.
7. 그 다음 기술을 선택한다.

## 12. 흔한 실수

### 실수 1: 바로 Kafka, Redis, Kubernetes부터 말한다
기술이 아니라 문제에서 출발해야 한다.

### 실수 2: 숫자를 너무 정밀하게 계산한다
System Design에서 estimation은 order of magnitude를 잡는 도구다.

### 실수 3: 평균만 본다
운영은 peak traffic에서 무너진다.

### 실수 4: 모든 요구사항을 최상급으로 잡는다
Strong consistency + ultra-low latency + global availability는 매우 비싸다.

## 13. Backend 연결

TIS나 일반 기업 시스템에서도 같은 사고를 쓸 수 있다.

예를 들어 TIFF→PDF API를 설계한다면:
- 파일 평균 크기
- 최대 페이지 수
- 동시 요청 수
- CPU/Memory 사용량
- 목표 처리 시간
- sync API인지 background job인지

이런 요구사항을 먼저 수치화해야 병렬화, queue, autoscaling 여부를 판단할 수 있다.

## 14. 60초 면접 답변

> System Design을 시작할 때는 먼저 functional requirement와 non-functional requirement를 분리합니다. 그 다음 DAU, RPS, peak traffic, read/write ratio, payload size, storage growth 같은 값을 거칠게 추정해서 어디가 병목이 될지 봅니다. 중요한 건 숫자를 정확히 맞히는 것이 아니라 order of magnitude를 통해 cache, queue, sharding 같은 기술이 정말 필요한지 판단하는 것입니다. 또한 latency, availability, consistency 요구사항은 서로 비용과 trade-off가 있기 때문에 모든 것을 최대로 잡지 않고 비즈니스 요구에 맞는 수준을 정해야 합니다.
