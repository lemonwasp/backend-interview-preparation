# 04. Process Address Space

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 자신의 말로 설명할 수 있어야 합니다.

- Process Address Space란 무엇인가?
- Code, Data, Heap, Stack은 각각 무엇을 저장하는가?
- Heap과 Stack은 무엇이 다른가?
- 왜 Process마다 독립된 주소 공간이 필요한가?
- Virtual Memory가 왜 중요한가?

---

## 1. Process는 자기만의 메모리 지도를 가진다

Process가 실행되면 운영체제는 그 Process에게 독립된 메모리 공간이 있는 것처럼 보이게 합니다.

이를 **Process Address Space**라고 합니다.

```text
Process A
0x0000 ... 0xFFFF

Process B
0x0000 ... 0xFFFF
```

두 Process가 같은 주소를 사용하더라도 실제 물리 메모리의 같은 위치를 의미하는 것은 아닙니다.

운영체제와 MMU가 Virtual Address를 Physical Address로 변환하기 때문입니다.

핵심은 다음입니다.

> 각 Process는 자신만의 연속된 주소 공간을 가진 것처럼 보이지만, 실제 물리 메모리는 운영체제가 따로 관리한다.

---

## 2. 주소 공간은 왜 여러 영역으로 나뉠까?

대표적인 개념적 구조는 다음과 같습니다.

```text
높은 주소
┌──────────────────────┐
│ Stack                │
│        ↓             │
│                      │
│        ↑             │
│ Heap                 │
├──────────────────────┤
│ Data / BSS           │
├──────────────────────┤
│ Code (Text)          │
└──────────────────────┘
낮은 주소
```

실제 배치는 운영체제, 실행 형식, ASLR 등에 따라 달라질 수 있으므로 위 그림은 개념적 모델입니다.

---

## 3. Code 영역

실행할 기계어 명령이 들어 있는 영역입니다.

예를 들어 C#이나 C로 작성한 소스 코드는 빌드와 Runtime 과정을 거쳐 CPU가 실행할 수 있는 코드로 연결됩니다.

Code 영역은 일반적으로 프로그램 명령을 저장하며, 임의로 수정하지 못하도록 읽기 전용 권한이 적용될 수 있습니다.

왜 코드와 데이터를 구분할까요?

- 잘못된 메모리 쓰기로 실행 코드가 변하는 것을 막을 수 있다.
- 권한을 다르게 적용할 수 있다.
- 일부 코드는 여러 Process가 공유 가능한 형태로 매핑될 수 있다.

---

## 4. Data와 BSS 영역

전역 변수와 정적 변수 같은 장기간 유지되는 데이터가 저장되는 영역입니다.

개념적으로:

- Data: 초기값이 있는 전역/정적 데이터
- BSS: 초기값이 없거나 0으로 초기화되는 전역/정적 데이터

예:

```c
int initialized = 10;  // Data
int counter;           // BSS 개념
```

이 변수들은 함수 호출이 끝나도 사라지지 않습니다.

---

## 5. Stack

Stack은 함수 호출과 관련된 실행 정보를 저장하는 공간입니다.

대표적으로:

- 지역 변수
- 함수 인자 일부
- 반환 주소
- 호출 프레임

등이 들어갈 수 있습니다.

함수 호출을 상자 쌓기로 생각하면 쉽습니다.

```text
main()
  ↓
login()
  ↓
validate()
```

호출할 때 Stack Frame이 쌓이고 함수가 끝나면 제거됩니다.

```text
| validate frame |
| login frame    |
| main frame     |
```

Thread마다 Stack을 따로 가지는 이유도 여기서 이해할 수 있습니다.

Thread A와 Thread B가 서로 다른 함수 실행 흐름을 가지기 때문입니다.

---

## 6. Heap

Heap은 실행 중 동적으로 필요한 데이터를 저장하는 공간입니다.

객체, 동적으로 생성되는 데이터 구조 등이 대표적입니다.

C#에서는 다음과 같은 객체들이 관리 Heap에 배치될 수 있습니다.

```csharp
var user = new User();
var list = new List<int>();
```

다만 .NET의 Managed Heap은 CLR과 Garbage Collector가 관리하므로 전통적인 C의 `malloc/free` Heap과 관리 방식이 동일한 것은 아닙니다.

핵심 개념은:

> 실행 중 수명과 크기를 미리 정확히 알기 어려운 데이터를 동적으로 관리하는 공간이다.

---

## 7. Stack과 Heap 비교

| 항목 | Stack | Heap |
|---|---|---|
| 주 용도 | 함수 호출, 지역 실행 상태 | 동적 데이터, 객체 |
| 관리 단위 | Thread별 | Process 안에서 공유 가능 |
| 수명 | 호출 흐름에 따라 자동 정리 | Runtime/프로그램이 수명 관리 |
| 속도 특성 | 일반적으로 단순한 push/pop | 할당과 회수 관리가 더 복잡 |
| 위험 | Stack Overflow | Memory Leak, OOM, GC 부담 |

“Stack은 무조건 빠르고 Heap은 무조건 느리다”처럼 외우는 것은 좋지 않습니다.

실제 성능은 Runtime, 캐시, 객체 수명, 할당 패턴 등에 따라 달라집니다.

---

## 8. Virtual Memory는 왜 필요한가?

프로그램이 실제 물리 RAM 주소를 직접 사용한다면 문제가 많습니다.

- Process끼리 주소 충돌
- 메모리 보호 어려움
- 프로그램마다 물리 메모리 위치를 알아야 함
- 메모리 이동과 재배치가 어려움

Virtual Memory는 각 Process에 독립적인 주소 공간을 제공합니다.

```text
Virtual Address
      ↓
Page Table / MMU
      ↓
Physical Memory
```

덕분에 Process는 자신의 메모리만 있는 것처럼 동작합니다.

운영체제는 실제 물리 메모리를 페이지 단위로 배치하고 보호합니다.

---

## 9. 같은 Virtual Address가 어떻게 가능할까?

Process A와 Process B가 모두 `0x1000`이라는 주소를 사용한다고 해봅시다.

```text
Process A 0x1000 -> Physical Page X
Process B 0x1000 -> Physical Page Y
```

가상 주소는 같지만 각 Process의 Page Table이 다르므로 실제 물리 위치는 다를 수 있습니다.

이것이 Process 격리의 핵심 기반 중 하나입니다.

---

## 10. Page Fault는 무조건 오류일까?

아닙니다.

CPU가 어떤 Virtual Page에 접근했는데 현재 필요한 매핑이나 물리 페이지가 준비되지 않은 경우 Page Fault가 발생할 수 있습니다.

운영체제는 상황에 따라:

- 필요한 Page를 메모리에 준비하거나
- 파일에서 데이터를 가져오거나
- 잘못된 접근이면 Process를 종료할 수 있습니다.

즉 Page Fault는 항상 치명적 오류를 뜻하는 것이 아니라 Virtual Memory 동작의 정상적인 일부일 수도 있습니다.

---

## 11. 백엔드와 연결

백엔드 개발에서 주소 공간 이해가 중요한 이유는 다음과 같습니다.

### OOM

Process가 계속 메모리를 할당하고 회수되지 않으면 사용 가능한 메모리가 부족해질 수 있습니다.

### Memory Leak

사용하지 않는 객체나 자원을 계속 참조하면 GC가 회수하지 못해 메모리 사용량이 증가할 수 있습니다.

### Stack Overflow

재귀 호출이 너무 깊어지면 Thread Stack이 고갈될 수 있습니다.

```text
func()
  -> func()
      -> func()
          -> ...
```

### GC

C#의 객체는 CLR Managed Heap에서 관리되고, Garbage Collector가 도달할 수 없는 객체를 찾아 회수합니다.

OS의 Virtual Memory와 CLR의 Managed Heap은 같은 개념이 아닙니다.

```text
OS Virtual Address Space
        ↓
Process Memory
        ↓
CLR Managed Heap
        ↓
C# Objects
```

---

## 12. TIFF-to-PDF 사례와 연결

이미지를 처리할 때 파일 I/O를 줄이고 `MemoryStream`을 사용하면 디스크 경로는 줄어들지만 메모리 사용량은 증가할 수 있습니다.

즉 성능 최적화에는 항상 Trade-off가 있습니다.

```text
Temporary File 방식
장점: 메모리 압박을 줄일 수 있음
단점: File System I/O 비용

MemoryStream 방식
장점: 불필요한 File I/O 감소
단점: Process 메모리 사용 증가 가능
```

따라서 큰 TIFF를 수백 장 동시에 처리한다면 처리 속도뿐 아니라 Peak Memory와 OOM 가능성도 확인해야 합니다.

---

## 13. 자주 하는 오해

### “Virtual Memory는 RAM이 부족할 때 디스크를 쓰는 기능이다”

그것만을 의미하지 않습니다. 주소 공간 추상화, 보호, 매핑이 더 근본적인 개념입니다.

### “Heap은 운영체제가 GC한다”

C# Managed Heap의 객체 수명 관리는 CLR의 Garbage Collector가 담당합니다. OS 메모리 관리와 구분해야 합니다.

### “모든 지역 변수는 무조건 Stack에 있다”

언어와 Runtime 최적화에 따라 실제 배치는 달라질 수 있습니다. 면접에서는 개념적 모델과 실제 Runtime 구현을 구분하는 것이 안전합니다.

### “Page Fault는 프로그램 오류다”

정상적인 Demand Paging에서도 발생할 수 있습니다.

---

## 14. 60초 면접 답변

> Process Address Space는 하나의 Process가 사용할 수 있는 가상 메모리 주소 범위입니다. 운영체제와 MMU는 Virtual Address를 Physical Memory에 매핑해 각 Process가 독립된 메모리를 가진 것처럼 보이게 합니다. 개념적으로 Code, Data, Heap, Stack 영역으로 나눌 수 있고, Stack은 함수 호출과 Thread별 실행 상태를, Heap은 실행 중 동적으로 필요한 객체와 데이터를 저장합니다. Virtual Memory 덕분에 Process 간 주소 충돌을 막고 메모리를 보호할 수 있습니다. 백엔드에서는 OOM, Memory Leak, Stack Overflow, GC와 같은 문제를 이해할 때 이 구조가 중요합니다.

---

## 핵심 요약

```text
Process Address Space = Process가 보는 독립적인 Virtual Memory 공간
Code  = 실행 명령
Data  = 전역/정적 데이터
Heap  = 동적 데이터
Stack = 함수 호출과 Thread별 실행 상태

Virtual Address -> Page Table/MMU -> Physical Memory
```

다음 주제:

- Context Switching
- CPU Scheduling
- Cache와 TLB 비용
