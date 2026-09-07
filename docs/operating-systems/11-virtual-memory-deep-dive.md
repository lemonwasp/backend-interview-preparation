# 11. Virtual Memory 심화

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 설명할 수 있어야 합니다.

- Virtual Address와 Physical Address가 왜 분리되어 있는가?
- Page Table과 MMU는 어떤 역할을 하는가?
- TLB는 왜 필요한가?
- Process마다 독립된 Address Space를 제공하면 어떤 장점이 있는가?
- Virtual Memory가 실제 RAM보다 큰 메모리를 쓸 수 있게 하는 것 이상의 의미를 갖는 이유는 무엇인가?

---

## 1. Virtual Memory의 핵심은 '주소 번역'

프로그램은 보통 자신이 연속된 거대한 메모리 공간을 독점해서 쓰는 것처럼 동작합니다.

```text
Process A Virtual Address
0x1000 ─────┐
            ├─ Page Table ─→ Physical Memory
0x2000 ─────┘
```

하지만 실제 RAM에서는 여러 Process의 Page가 섞여 있을 수 있습니다.

운영체제와 CPU는 Virtual Address를 Physical Address로 번역해 이 차이를 감춥니다.

---

## 2. 왜 굳이 Virtual Address를 쓸까?

### 격리

Process A가 보는 `0x1000`과 Process B가 보는 `0x1000`은 서로 다른 Physical Page로 연결될 수 있습니다.

따라서 각 Process는 독립된 메모리 공간을 가진 것처럼 동작합니다.

### 단순한 프로그래밍 모델

프로그램은 실제 RAM의 어느 위치에 배치될지 직접 관리하지 않아도 됩니다.

### 보호

Page마다 읽기/쓰기/실행 권한을 설정할 수 있습니다.

### 유연한 메모리 관리

필요한 Page만 RAM에 올리고, 파일 매핑이나 Copy-on-Write 같은 기법도 사용할 수 있습니다.

---

## 3. Page Table

Virtual Memory는 보통 고정 크기 단위인 Page로 관리됩니다.

```text
Virtual Page Number + Offset
          ↓
      Page Table
          ↓
Physical Frame Number + Offset
```

Page Table은 Process마다 Virtual Page가 어느 Physical Frame에 매핑되는지 기록합니다.

---

## 4. MMU

MMU(Memory Management Unit)는 CPU가 메모리 주소를 사용할 때 Virtual → Physical 변환을 수행하는 하드웨어입니다.

개념적으로:

```text
CPU Virtual Address
       ↓
      MMU
       ↓
TLB / Page Table
       ↓
Physical Address
```

운영체제가 Page Table을 관리하고, MMU가 실제 주소 변환을 빠르게 수행합니다.

---

## 5. TLB는 왜 필요한가?

매 메모리 접근마다 Page Table을 메모리에서 다시 찾아야 한다면 너무 느립니다.

그래서 CPU에는 최근 주소 변환 결과를 저장하는 작은 Cache인 **TLB(Translation Lookaside Buffer)**가 있습니다.

```text
Virtual Address
   ↓
TLB hit  → 빠르게 Physical Address 획득
TLB miss → Page Table 확인 후 TLB 갱신
```

Context Switch나 큰 Working Set은 TLB 효율에 영향을 줄 수 있습니다.

---

## 6. Virtual Memory = Swap만은 아니다

자주 하는 오해입니다.

Virtual Memory를 단순히

> RAM이 부족하면 디스크를 RAM처럼 쓰는 기술

이라고만 이해하면 부족합니다.

더 중요한 핵심은:

- 주소 공간 추상화
- Process 격리
- 보호
- Demand Paging
- File Mapping
- Copy-on-Write

입니다.

Swap은 Virtual Memory 시스템에서 사용할 수 있는 여러 메커니즘 중 하나입니다.

---

## 7. Copy-on-Write

두 Process가 처음에는 같은 Physical Page를 읽기 전용처럼 공유하다가, 한쪽이 수정하려 할 때 실제 복사를 만드는 방식입니다.

```text
Process A ─┐
           ├─ Shared Physical Page
Process B ─┘

Process B writes
      ↓
copy page
      ↓
A → original
B → copied page
```

불필요한 복사를 줄이는 데 유용합니다.

---

## 8. Memory-mapped File

파일의 일부를 Process의 Virtual Address Space에 매핑할 수도 있습니다.

애플리케이션 입장에서는 메모리 접근처럼 보이지만 운영체제가 필요 시 해당 파일 Page를 가져옵니다.

이 방식은 대용량 파일, DB, 공유 메모리 등의 구현에서 사용될 수 있습니다.

---

## 9. 백엔드 관점

Virtual Memory를 알면 다음 현상이 더 잘 보입니다.

- OOM인데 실제 RAM 사용량만 보면 이상한 경우
- Process RSS와 Virtual Size 차이
- Page Fault 증가
- Memory-mapped DB 파일
- Container Memory Limit
- GC와 OS Memory Pressure 관계

Runtime이 관리하는 Heap도 결국 OS의 Virtual Memory 위에서 동작합니다.

---

## 10. 60초 면접 답변

> Virtual Memory는 각 Process에 독립된 주소 공간을 제공하고, Virtual Address를 실제 Physical Memory에 매핑하는 메커니즘입니다. 운영체제가 Page Table을 관리하고 CPU의 MMU가 주소 변환을 수행하며, TLB가 최근 변환을 Cache해서 성능을 높입니다. 이를 통해 Process 간 메모리 격리와 보호가 가능하고, 필요한 Page만 RAM에 올리는 Demand Paging이나 Memory-mapped File, Copy-on-Write 같은 기능도 구현할 수 있습니다. 그래서 Virtual Memory는 단순히 RAM이 부족할 때 디스크를 쓰는 기술보다 훨씬 넓은 개념입니다.

---

## 핵심 요약

```text
Virtual Address → MMU/TLB/Page Table → Physical Address
Virtual Memory = 주소 추상화 + 격리 + 보호 + 유연한 Page 관리
```

다음 주제: Paging과 Page Fault
