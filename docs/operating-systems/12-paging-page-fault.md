# 12. Paging과 Page Fault

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 설명할 수 있어야 합니다.

- Page와 Frame은 무엇이 다른가?
- Page Fault는 왜 반드시 오류를 의미하지 않는가?
- Minor Page Fault와 Major Page Fault는 어떻게 다른가?
- Demand Paging은 왜 필요한가?
- Working Set이 너무 커지면 왜 성능이 급격히 나빠질 수 있는가?

---

## 1. Page와 Frame

Virtual Memory는 보통 일정한 크기의 **Page** 단위로 관리됩니다.
Physical Memory의 대응 단위를 **Frame**이라고 생각하면 됩니다.

```text
Virtual Page 0 ─→ Physical Frame 5
Virtual Page 1 ─→ Physical Frame 2
Virtual Page 2 ─→ not resident
```

Program은 Virtual Page를 사용하고, 운영체제는 필요한 Page를 Physical Frame에 배치합니다.

---

## 2. Demand Paging

프로그램 실행 시 모든 코드와 데이터를 한꺼번에 RAM에 올리는 것은 낭비일 수 있습니다.

Demand Paging은 실제로 필요한 Page를 접근할 때 RAM에 준비하는 방식입니다.

```text
program starts
   ↓
only some pages resident
   ↓
access missing page
   ↓
Page Fault
   ↓
OS prepares page
   ↓
execution resumes
```

따라서 Page Fault 자체는 정상적인 메모리 관리 과정일 수 있습니다.

---

## 3. Page Fault란?

CPU가 어떤 Virtual Page에 접근했는데 현재 Page Table 상태상 바로 접근할 수 없으면 예외가 발생하고 Kernel이 개입합니다.

Kernel은 원인을 확인합니다.

### 정상적으로 처리 가능한 경우

- 아직 RAM에 올리지 않은 Page
- Copy-on-Write로 복사가 필요한 Page
- Memory-mapped File의 아직 준비되지 않은 Page

### 실제 오류인 경우

- 접근 권한이 없는 주소
- 존재하지 않는 Mapping

후자의 경우 Process가 종료되는 식의 오류로 이어질 수 있습니다.

즉:

> Page Fault = 무조건 프로그램 버그

가 아닙니다.

---

## 4. Minor vs Major Page Fault

운영체제별 세부 정의는 다를 수 있지만 개념적으로 다음처럼 이해할 수 있습니다.

### Minor Page Fault

필요한 데이터가 이미 메모리 어딘가에 있어 물리 디스크 I/O 없이 Mapping 조정 등으로 해결할 수 있는 경우입니다.

### Major Page Fault

필요한 Page를 Storage에서 읽어와야 해 실제 I/O 대기가 필요한 경우입니다.

Major Fault는 일반적으로 훨씬 비쌉니다.

---

## 5. Page Fault 처리 흐름

```text
CPU accesses virtual address
        ↓
Page not currently usable
        ↓
Page Fault trap
        ↓
Kernel checks mapping/permission
        ↓
valid? ─ no → error
  |
 yes
  ↓
prepare/load page
  ↓
update Page Table / TLB
  ↓
retry instruction
```

---

## 6. Working Set

Working Set은 한 Process가 현재 활발히 사용 중인 Page들의 집합이라고 생각하면 됩니다.

Working Set이 RAM 안에 잘 들어가면 메모리 접근이 비교적 안정적입니다.

하지만 여러 Process의 Working Set 합이 Physical Memory보다 너무 커지면 Page 교체가 잦아질 수 있습니다.

---

## 7. Thrashing

Page를 계속 가져오고 내보내느라 실제 유용한 계산보다 Paging 작업에 더 많은 시간을 쓰는 상태를 Thrashing이라고 합니다.

```text
need page A → load A
need page B → evict A, load B
need page A → evict B, load A
...
```

이 상태에서는 CPU 사용률보다 I/O와 Page Fault가 병목이 될 수 있습니다.

---

## 8. Page Replacement

RAM이 꽉 찼을 때 어떤 Page를 내보낼지 선택해야 합니다.

운영체제는 최근 사용 정보 등을 바탕으로 근사적인 LRU 계열 전략을 사용할 수 있습니다.
정확한 알고리즘은 OS 구현에 따라 다릅니다.

핵심은:

> 자주 다시 쓸 Page를 계속 내보내면 성능이 급격히 나빠질 수 있다.

입니다.

---

## 9. 백엔드에서의 연결

### 큰 in-memory Cache

Cache를 크게 잡으면 DB 요청은 줄 수 있지만 Process Working Set이 커지고 Memory Pressure가 증가할 수 있습니다.

### 대용량 파일 처리

파일 전체를 한 번에 메모리에 올리면 Working Set이 커질 수 있습니다.
Streaming이나 Chunk 처리가 더 안정적일 수 있습니다.

### GC

Managed Heap이 커질수록 OS 수준에서는 더 많은 Page가 필요해지고 Memory Pressure와 Page Fault 특성도 달라질 수 있습니다.

### Container

Container Memory Limit에 가까워질수록 Runtime Heap만 보는 것으로 충분하지 않을 수 있습니다.

---

## 10. TIFF-to-PDF 사례

`MemoryStream`으로 임시 파일을 제거하면 File System I/O를 줄일 수 있지만 메모리 사용량이 늘어날 수 있습니다.

따라서 매우 큰 TIFF를 동시에 여러 개 처리하면:

```text
less temp-file I/O
        but
larger working set
        ↓
Memory Pressure / GC / Page Fault risk
```

가 될 수 있습니다.

성능 최적화는 CPU, Disk, Memory 중 비용을 어디로 옮기는지까지 봐야 합니다.

---

## 11. 60초 면접 답변

> Paging은 Virtual Memory를 Page 단위로 나누고 이를 Physical Memory의 Frame에 매핑해서 관리하는 방식입니다. 프로그램이 현재 RAM에 없는 Page에 접근하면 Page Fault가 발생하고 Kernel이 해당 Mapping과 권한을 확인한 뒤 필요한 Page를 준비합니다. 따라서 Page Fault 자체는 정상적인 Demand Paging 과정일 수 있습니다. 다만 Storage I/O까지 필요한 Major Page Fault는 비용이 크고, Working Set이 RAM보다 커져 Page 교체가 계속 발생하면 Thrashing으로 성능이 크게 저하될 수 있습니다.

---

## 핵심 요약

```text
Page  = Virtual Memory 관리 단위
Frame = Physical Memory 대응 단위
Page Fault ≠ 항상 오류
Major Fault = Storage I/O 가능 → 비쌈
Working Set이 RAM을 압박하면 Thrashing 위험
```

다음 주제: Page Cache와 File I/O
