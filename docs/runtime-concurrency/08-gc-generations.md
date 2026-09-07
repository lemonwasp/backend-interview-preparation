# 08. GC Generations

## 한 줄 요약

.NET GC는 모든 객체를 매번 똑같이 검사하지 않고, **대부분의 객체는 빨리 죽고 일부만 오래 산다**는 관찰을 이용해 Generation별로 관리한다.

## 파인만식 설명

객체를 회사 직원이라고 생각하자.

- 막 들어온 단기 알바: 곧 나갈 가능성이 높음
- 오래 근무한 직원: 앞으로도 남아 있을 가능성이 높음

GC는 이런 특성을 이용해 새 객체를 자주 검사하고 오래 살아남은 객체는 덜 자주 검사한다.

## Managed Heap과 Generation

.NET의 일반적인 세대 구분:

- Gen 0: 새로 생성된 객체
- Gen 1: Gen 0에서 살아남은 중간 단계
- Gen 2: 오래 살아남은 객체

새 객체는 일반적으로 Gen 0에 들어간다.

Gen 0 GC에서 살아남으면 상위 Generation으로 승격될 수 있다.

## 왜 Generation을 나누나?

대부분의 Backend 요청에서 만들어지는 임시 객체는 짧게 산다.

예:

```text
HTTP Request
→ DTO
→ temporary List
→ JSON serialization buffer
→ response
```

요청이 끝나면 많은 객체가 더 이상 필요하지 않다.

따라서 매번 전체 Heap을 검사하는 대신 젊은 객체 영역을 빠르게 수집하면 효율적이다.

## GC Root와 Reachability

GC는 단순히 "오래된 객체"를 지우는 것이 아니다.

핵심은 **도달 가능한가**이다.

대표적인 Root 개념:

- Thread Stack의 reference
- Static field
- Runtime handle
- 일부 native interop reference

Root에서 reference chain을 따라 도달할 수 있는 객체는 살아 있는 것으로 본다.

## Stop-the-world

GC 과정의 일부 단계에서는 managed Thread 실행이 일시 정지될 수 있다.

이 pause가 매우 길거나 자주 발생하면 latency에 영향을 줄 수 있다.

하지만 "GC가 실행되면 서버 전체가 몇 초씩 멈춘다"처럼 일반화하면 안 된다. GC mode, heap size, allocation rate, generation, workload에 따라 다르다.

## Server GC vs Workstation GC

.NET에는 workload에 따라 다른 GC mode가 있다.

Backend 서버에서는 Server GC가 사용되는 경우가 많다. 여러 heap/worker를 활용해 throughput을 높이는 방향의 특성을 가진다.

정확한 동작은 Runtime 버전과 설정에 따라 달라질 수 있으므로 면접에서는 세부 구현을 과도하게 단정하지 않는다.

## Large Object Heap

큰 객체는 일반 작은 객체와 다른 경로로 관리될 수 있다.

.NET에서는 일반적으로 약 85,000 bytes 이상 크기의 객체가 LOH(Large Object Heap)에 들어가는 것으로 알려져 있다.

예:

```csharp
byte[] buffer = new byte[10_000_000];
```

큰 byte array, image buffer, large string/array가 반복 생성되면 memory pressure와 GC 비용을 키울 수 있다.

## TIFF→PDF 사례 연결

이미지 처리에서는 다음이 중요하다.

```text
TIFF decode
→ RGBA buffer
→ Bitmap
→ MemoryStream
→ PDF
```

디스크 I/O를 없애기 위해 MemoryStream을 쓰는 것은 성능상 유리할 수 있지만, 큰 이미지 여러 장을 동시에 처리하면 Managed/Unmanaged memory pressure가 커질 수 있다.

즉:

```text
Disk I/O 감소
↔ Memory usage 증가
```

라는 trade-off가 있다.

## Promotion과 오래 사는 객체

객체가 계속 살아남으면 상위 Generation으로 승격된다.

문제는 실제로 필요해서 오래 사는 객체와, 실수로 reference가 남아 있어 오래 사는 객체를 구분해야 한다는 점이다.

예:

- static cache가 계속 커짐
- event handler unsubscribe 누락
- timer callback이 객체를 잡고 있음

이런 경우 객체가 계속 reachable해 GC가 수집할 수 없다.

## GC가 있다고 Memory Leak이 없는 것은 아니다

GC는 **도달 불가능한 객체**를 자동 회수한다.

하지만 필요 없는 객체가 여전히 reachable하면 GC는 그것을 정상적인 live object로 본다.

그래서 managed environment에도 logical memory leak이 존재한다.

## Allocation Rate가 중요한 이유

메모리 사용량이 일정해 보여도 초당 allocation이 매우 많으면 Gen 0 GC가 자주 발생할 수 있다.

따라서 Runtime observability에서는 단순 RSS/Heap size뿐 아니라:

- allocation rate
- Gen 0/1/2 collection count
- pause time
- LOH size
- GC CPU time

등을 함께 본다.

## 흔한 오해

### "GC면 메모리 관리 신경 안 써도 된다"

아니다. allocation pattern과 reference lifetime이 성능에 직접 영향을 준다.

### "Gen 2 객체는 절대 안 지워진다"

아니다. 오래 살아남았다는 뜻이지 영구 객체라는 뜻이 아니다.

### "GC는 메모리가 부족할 때만 실행된다"

아니다. Runtime이 allocation pressure와 여러 조건을 보고 수행한다.

## 60초 면접 답변

> .NET GC는 대부분의 객체가 짧게 산다는 가정을 이용해 Gen 0, Gen 1, Gen 2로 Heap을 나눠 관리합니다. 새 객체는 주로 Gen 0에서 시작하고 GC에서 살아남으면 상위 세대로 승격될 수 있습니다. GC는 Root에서 도달 가능한 객체를 live로 판단하기 때문에 GC가 있다고 memory leak이 사라지는 것은 아니고, static cache나 event reference처럼 불필요한 객체가 계속 reachable하면 메모리가 증가할 수 있습니다. Backend에서는 Heap 크기뿐 아니라 allocation rate, Gen 0/1/2 collection, pause time, LOH 같은 지표를 함께 봐야 합니다. 큰 image buffer나 MemoryStream을 많이 만드는 작업은 disk I/O를 줄이는 대신 memory pressure와 GC 비용을 키울 수 있습니다.
