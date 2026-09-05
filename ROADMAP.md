# Backend Interview Roadmap

기술면접에 필요한 순서대로 학습합니다. 모든 항목을 같은 깊이로 공부하지 않고,
백엔드 개발에서 자주 연결되는 개념을 우선합니다.

## Phase 1. Operating Systems

| No. | Topic | Expected output |
|---:|---|---|
| 01 | OS and Kernel | OS와 Kernel의 역할을 구분해 설명 |
| 02 | User Mode, Kernel Mode, System Calls | 애플리케이션이 하드웨어를 사용하는 과정 설명 |
| 03 | Program and Process | Process 생성과 상태, PCB 설명 |
| 04 | Process and Thread | 공유 자원과 Context Switching 비교 |
| 05 | CPU Scheduling | Scheduling이 필요한 이유와 주요 방식 설명 |
| 06 | Stack, Heap, Virtual Memory | 주소 공간과 메모리 보호 설명 |
| 07 | Paging, TLB, Page Fault | 가상 주소가 실제 메모리로 연결되는 과정 설명 |
| 08 | Race Condition and Atomicity | 동시성 오류를 코드로 재현 |
| 09 | Mutex, Semaphore, Deadlock | 동기화 도구와 Deadlock 조건 설명 |
| 10 | Sync, Async, Blocking, Non-blocking | 네 개념을 서로 다른 축으로 구분 |
| 11 | File System, Buffer, Cache, File Descriptor | 파일 I/O의 실제 경로 설명 |
| 12 | OS Interview Review | OS 모의면접 및 취약점 보강 |

## Phase 2. Networking

- OSI와 TCP/IP 모델
- IP, Port, Socket
- TCP와 UDP
- TCP 연결 및 종료
- 흐름 제어와 혼잡 제어
- DNS
- HTTP와 HTTPS
- HTTP/1.1, HTTP/2, HTTP/3
- Cookie, Session, Token
- Proxy, Reverse Proxy, Load Balancer
- Timeout, Retry, Idempotency
- 네트워크 모의면접

## Phase 3. Databases

- 관계형 데이터베이스와 테이블
- Primary Key와 Foreign Key
- B-Tree와 Index
- Query 실행 계획
- Transaction과 ACID
- Isolation Level
- Lock과 MVCC
- 정규화와 반정규화
- Join
- Replication
- Partitioning과 Sharding
- 데이터베이스 모의면접

## Phase 4. Runtime and Backend Concurrency

- 프로세스, Thread Pool, Event Loop
- CPU-bound와 I/O-bound
- 동기 I/O와 비동기 I/O
- Garbage Collection
- Connection Pool
- Backpressure
- 동시성 제어
- 장애와 자원 고갈 분석

## Phase 5. System Design

- 요구사항과 규모 추정
- API와 데이터 모델
- Stateless Server
- Cache
- Message Queue
- Load Balancing
- Database Replication
- Sharding
- Consistency와 Availability
- Rate Limiting
- Observability
- 장애 격리와 복구
- 대표 시스템 설계 문제

## Weekly Cycle

```text
쉬운 설명
  → 자신의 말로 재설명
  → 확인 질문
  → 코드/도구 실험
  → 면접 답변
  → 피드백과 문서 보강
  → 1일·7일 후 재시험
```

## Priority

현재 우선순위는 다음과 같습니다.

1. Python 코딩테스트를 매일 유지한다.
2. OS·네트워크·DB를 주 3회 학습한다.
3. 기본기가 만들어지면 시스템 설계를 시작한다.
4. 각 영역의 심화 내용은 모의면접에서 실제로 부족한 부분만 확장한다.
