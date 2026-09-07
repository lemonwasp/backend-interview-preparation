# 13. Page Cache와 File I/O

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 설명할 수 있어야 합니다.

- File I/O가 항상 즉시 Physical Disk까지 내려가는 것은 아닌 이유는 무엇인가?
- Page Cache는 무엇을 하는가?
- Buffered I/O와 Direct I/O의 차이는 무엇인가?
- `write()`가 성공했다고 해서 데이터가 즉시 Storage에 영구 저장되었다고 단정할 수 없는 이유는 무엇인가?
- `fsync` 계열 동작이 왜 비용이 큰가?
- Sequential I/O와 Random I/O는 왜 성능 특성이 다른가?

---

## 1. 파일을 읽는다고 항상 Disk를 읽는 것은 아니다

애플리케이션이 파일을 읽으면 보통 Kernel의 File System 계층을 거칩니다.

```text
Application
   ↓ read
Kernel File System
   ↓
Page Cache hit? ─ yes → memory에서 반환
   |
   no
   ↓
Storage I/O
   ↓
Page Cache에 적재
   ↓
Application에 반환
```

최근 읽은 파일 데이터가 Page Cache에 남아 있다면 Physical Storage 접근 없이 메모리에서 빠르게 반환될 수 있습니다.

---

## 2. Page Cache란?

운영체제가 파일 데이터를 RAM에 Cache해 두는 영역입니다.

목적은 간단합니다.

> Storage는 RAM보다 훨씬 느리므로 자주 쓰는 파일 데이터를 메모리에 남겨 재사용한다.

Page Cache는 File I/O와 Virtual Memory 시스템이 만나는 중요한 지점입니다.

---

## 3. Read 흐름

### Cache Hit

```text
read()
  ↓
requested page already in Page Cache
  ↓
copy/return data
```

### Cache Miss

```text
read()
  ↓
Page Cache miss
  ↓
Storage request
  ↓
wait for I/O
  ↓
cache page
  ↓
return data
```

같은 파일을 두 번째 읽었을 때 훨씬 빨라지는 이유 중 하나가 Page Cache입니다.

---

## 4. Write 흐름

일반적인 Buffered Write에서는 애플리케이션이 쓴 데이터가 먼저 Kernel Buffer/Page Cache에 반영되고, 실제 Storage 반영은 나중에 이루어질 수 있습니다.

```text
Application write
      ↓
Page Cache dirty page
      ↓
write() returns
      ↓
Kernel flushes later
      ↓
Storage
```

따라서 `write()` 성공과 **durable storage 반영**은 같은 의미가 아닐 수 있습니다.

---

## 5. Dirty Page와 Flush

수정됐지만 아직 Storage에 반영되지 않은 Cache Page를 Dirty Page라고 부릅니다.

운영체제는 적절한 시점에 Dirty Page를 Background Writeback으로 Storage에 내릴 수 있습니다.

하지만 전원 장애나 시스템 Crash 상황에서도 데이터가 반드시 살아남아야 하는 경우에는 더 강한 동기화가 필요합니다.

---

## 6. fsync는 왜 비쌀까?

`fsync` 계열 동작은 단순히 Memory Buffer에 썼다고 끝내는 것이 아니라, 해당 데이터가 Storage 쪽에 안정적으로 반영되도록 기다리는 의미를 가질 수 있습니다.

그래서 요청마다 강한 Flush를 수행하면 Latency와 Throughput에 큰 영향을 줄 수 있습니다.

DB가 WAL, Group Commit 같은 전략을 사용하는 이유도 이 비용과 관련이 있습니다.

---

## 7. Buffered I/O vs Direct I/O

### Buffered I/O

운영체제 Page Cache를 활용합니다.

장점:

- 반복 읽기 빠름
- OS가 Cache 관리
- 일반 애플리케이션에서 사용하기 편함

### Direct I/O

가능한 경우 Page Cache를 우회해 Storage I/O를 직접적으로 관리하려는 방식입니다.

DB 같은 시스템은 자체 Buffer Pool이 이미 있기 때문에 Double Caching을 줄이기 위해 Direct I/O 계열 방식을 고려할 수 있습니다.

단, 실제 지원 방식과 제약은 OS/File System에 따라 다릅니다.

---

## 8. Sequential vs Random I/O

Sequential I/O는 연속된 영역을 읽고 쓰는 패턴입니다.

Random I/O는 떨어진 위치를 자주 접근합니다.

HDD에서는 Head 이동 때문에 차이가 특히 컸고, SSD에서도 Access Pattern, Queueing, Controller 특성 때문에 완전히 같은 비용이라고 볼 수는 없습니다.

운영체제는 Sequential Access를 감지하면 Readahead로 다음 Page를 미리 읽기도 합니다.

---

## 9. 작은 I/O를 많이 하면 왜 비효율적일까?

다음 두 방식이 있다고 합시다.

```text
1 byte write × 1,000,000
vs
1 MB buffered write × 1
```

작은 요청을 지나치게 많이 만들면:

- System Call 횟수 증가
- Kernel bookkeeping 증가
- Context/Mode Transition 비용 증가
- Storage request fragmentation 가능성

등이 생길 수 있습니다.

그래서 Buffering과 Batching이 중요합니다.

---

## 10. TIFF-to-PDF 사례와 연결

기존 구현:

```text
TIFF page
  ↓
PNG temp file write
  ↓
PNG temp file read
  ↓
PDF add
  ↓
delete
```

파일이 Page Cache에 있었더라도 다음 비용이 존재할 수 있습니다.

- File create/delete metadata 처리
- File API/System Call
- 경로 조회
- Buffer 복사
- Cache 관리

따라서 `MemoryStream`으로 임시 파일 경로 자체를 없앤 것은 단순히 “물리 Disk 접근 제거”라고 표현하기보다:

> 불필요한 File System 경로와 File I/O 계층 상호작용을 줄였다.

라고 설명하는 편이 정확합니다.

---

## 11. DB와 Page Cache

DB는 자체 Buffer Pool을 사용하는 경우가 많습니다.

```text
DB Buffer Pool
      ↓
OS Page Cache
      ↓
Storage
```

구조에 따라 같은 데이터를 DB와 OS가 모두 Cache하는 Double Caching이 생길 수 있습니다.

그래서 DB Storage Engine은 OS와 File System 특성을 고려해 I/O 전략을 설계합니다.

---

## 12. 60초 면접 답변

> Page Cache는 운영체제가 File 데이터를 RAM에 Cache해서 반복적인 Storage I/O를 줄이는 메커니즘입니다. 파일을 읽을 때 Cache hit이면 실제 Storage 접근 없이 메모리에서 반환할 수 있고, write도 Buffered I/O라면 먼저 Dirty Page로 반영한 뒤 Storage에는 나중에 Flush될 수 있습니다. 그래서 `write()` 성공이 곧 영구 저장 완료를 의미하지는 않을 수 있고, durability가 필요하면 fsync 같은 동작이 중요해집니다. 반면 fsync는 Storage 반영을 기다릴 수 있어서 비싸기 때문에 DB는 WAL이나 Group Commit 같은 전략으로 비용을 줄입니다.

---

## 핵심 요약

```text
File I/O ≠ 항상 Physical Disk I/O
Page Cache = File Data의 RAM Cache
write success ≠ durability guarantee
fsync = durability 강화, 대신 비용 큼
Buffering/Batching = System Call과 I/O 효율 개선
```

다음 주제: I/O Multiplexing
