# 10. Managed Memory Leak

## 한 줄 요약

GC가 있어도 Memory Leak은 생긴다. 문제는 메모리를 free하지 않는 것이 아니라, **더 이상 필요 없는 객체가 계속 reachable해서 GC가 정상적인 live object로 판단하는 것**이다.

## 파인만식 설명

창고 정리 담당자인 GC가 있다고 하자.

GC는 "아무도 사용하지 않는 물건"은 버릴 수 있다.

그런데 누군가 필요 없는 물건에 계속 이름표를 붙여 놓으면 GC는 버리면 안 되는 물건이라고 판단한다.

Managed Memory Leak도 비슷하다.

```text
객체는 더 이상 비즈니스적으로 필요 없음
하지만 reference는 남아 있음
→ GC Root에서 계속 도달 가능
→ 수집되지 않음
→ Heap 증가
```

## 대표 원인 1: Static Collection

```csharp
static readonly List<User> Users = new();
```

요청이 올 때마다 계속 추가하고 제거하지 않으면 모든 객체가 static root를 통해 계속 reachable하다.

GC 관점에서는 정상적인 live object다.

## 대표 원인 2: Event Handler

```csharp
publisher.Event += subscriber.Handle;
```

Publisher가 오래 사는 객체이고 Subscriber가 짧게 살아야 하는데 unsubscribe하지 않으면 publisher → delegate → subscriber reference chain이 남을 수 있다.

```text
Long-lived Publisher
→ Event Delegate
→ Subscriber
```

Subscriber가 GC되지 못한다.

## 대표 원인 3: Timer / Callback

`Timer`, scheduled callback, background worker가 closure나 instance reference를 계속 잡고 있으면 객체 lifetime이 예상보다 길어질 수 있다.

## 대표 원인 4: Unbounded Cache

Cache가 TTL/size limit/eviction 없이 계속 커지면 이것도 사실상 memory leak처럼 동작한다.

```text
cache miss
→ add
→ never evict
→ memory keeps growing
```

따라서 Cache는 "빠른 Dictionary"가 아니라 **lifetime policy가 있는 저장소**여야 한다.

## 대표 원인 5: Unbounded Queue

Producer가 Consumer보다 빠른데 queue limit가 없으면 backlog가 memory에 계속 쌓인다.

이 경우 객체가 누수된 것은 아닐 수 있지만 운영상 결과는 비슷하다.

```text
incoming rate > processing rate
→ queue length 증가
→ retained objects 증가
→ memory 증가
```

이건 Backpressure 문제와도 연결된다.

## 대표 원인 6: HttpContext / Request Object 보관

ASP.NET 요청 범위 객체를 static/cache/background task에 오래 보관하면 request graph 전체를 붙잡을 수 있다.

특히 큰 body, user context, scoped service와 연결되면 retention 규모가 커질 수 있다.

## 대표 원인 7: IDisposable / Native Resource

이건 managed memory leak과 조금 다르다.

`FileStream`, Socket, DB Connection, native handle 같은 resource는 GC 대상 object와 OS/native resource가 연결되어 있을 수 있다.

`Dispose()`를 제때 하지 않으면:

- FD/handle leak
- connection exhaustion
- unmanaged memory retention

이 생길 수 있다.

즉:

```text
Managed memory leak
!=
Resource leak
```

하지만 실무에서는 함께 나타날 수 있다.

## Memory Leak과 High Allocation Rate 구분

### High Allocation Rate

객체를 많이 만들지만 대부분 잘 수집됨.

특징:

- allocation/sec 높음
- GC 자주 발생
- heap size는 일정 범위에서 유지될 수도 있음

### Memory Leak / Retention

필요 없는 객체가 계속 살아남음.

특징:

- Gen 2 live data 증가
- heap baseline이 계속 상승
- GC 후에도 memory가 충분히 내려오지 않음

둘은 원인과 해결법이 다르다.

## RSS와 Managed Heap이 다를 수 있다

Process RSS가 높다고 바로 Managed Memory Leak이라고 단정하면 안 된다.

Process memory에는 다음도 포함될 수 있다.

- Managed Heap
- Thread Stack
- JIT Code
- Native Library
- Unmanaged Buffer
- Memory-mapped region
- Runtime metadata

따라서 먼저 **어느 memory가 증가하는지** 분리해야 한다.

## 진단 순서

### 1. 추세 확인

- Process RSS
- GC Heap size
- Gen 2 size
- LOH size
- Allocation Rate
- GC Count / Pause

### 2. Heap Dump 비교

한 번의 snapshot보다 시간 간격을 둔 비교가 유용하다.

```text
T1 heap dump
→ traffic 지속
→ T2 heap dump
→ 어떤 Type의 instance/retained size가 증가했는지 비교
```

### 3. Retention Path 확인

중요한 질문은:

> "이 객체를 누가 잡고 있어서 GC Root까지 연결되는가?"

이다.

단순 instance count보다 root path가 원인 찾기에 더 중요하다.

## .NET 진단 도구 예

- `dotnet-counters`
- `dotnet-gcdump`
- `dotnet-dump`
- Visual Studio Diagnostic Tools
- PerfView
- 상용 profiler

면접에서는 도구 이름보다 **진단 논리**를 말하는 것이 중요하다.

## Backend 사례

### 사례 1: Static Dictionary Cache

증상:

- 트래픽과 함께 memory baseline 상승
- Gen 2 object 증가

원인:

- eviction 없는 static cache

해결:

- size limit
- TTL
- eviction
- bounded cache

### 사례 2: Background Queue

증상:

- memory와 queue length가 함께 증가

원인:

- consumer capacity 부족
- unbounded queue

해결:

- bounded Channel
- concurrency 조정
- load shedding
- producer backpressure

### 사례 3: Event Subscription

증상:

- 특정 request/scoped object가 계속 남음

원인:

- long-lived singleton publisher에 subscriber 등록 후 unsubscribe 누락

## Finalizer와 Dispose

Finalizer가 있다고 `Dispose()`를 생략해도 된다는 뜻은 아니다.

Finalization은 시점이 비결정적이고 GC 비용을 늘릴 수 있다.

외부 resource lifetime은 `using`/`Dispose()`로 명시적으로 관리하는 것이 기본이다.

## 흔한 오해

### "GC가 있으면 Memory Leak은 없다"

틀렸다. reachable한 불필요 객체는 GC가 지우지 않는다.

### "메모리가 높으면 무조건 Leak이다"

틀렸다. 정상 cache, GC heap reservation, native memory, high allocation 등 다른 원인이 있을 수 있다.

### "GC.Collect()를 호출하면 Leak이 해결된다"

아니다. Root reference가 남아 있으면 강제 GC를 해도 객체는 살아남는다. 오히려 성능만 악화시킬 수 있다.

## 60초 면접 답변

> Managed 환경에서도 memory leak은 발생할 수 있습니다. GC는 도달 불가능한 객체만 수집하기 때문에, 더 이상 필요 없는 객체가 static collection, event handler, timer, unbounded cache 같은 reference chain을 통해 GC Root에서 계속 reachable하면 live object로 남습니다. 진단할 때는 단순 Process RSS만 보지 않고 GC Heap, Gen 2, LOH, allocation rate를 함께 보고, 시간차 heap dump를 비교해 증가하는 Type과 retention path를 확인합니다. 또 managed leak과 high allocation rate, native resource leak을 구분해야 합니다. 해결은 GC를 강제로 돌리는 것이 아니라 불필요한 reference를 제거하고 cache/queue에 lifetime과 capacity 정책을 두는 것입니다.
