# 09. Allocation and Boxing

## 한 줄 요약

Backend에서 메모리 문제는 단순히 "얼마나 많이 들고 있나"뿐 아니라 **얼마나 빠르게 새 객체를 만들고 버리나**도 중요하다. 불필요한 allocation과 boxing은 GC 빈도와 CPU 비용을 키울 수 있다.

## 파인만식 설명

매 요청마다 종이컵을 수천 개 쓰고 바로 버리는 식당을 생각해보자.

가게 안에 쌓여 있는 컵의 개수가 많지 않더라도 계속 새 컵을 만들고 치우는 비용이 크다.

Managed Heap도 비슷하다.

```text
많은 temporary object 생성
→ allocation rate 증가
→ GC 더 자주 수행
→ CPU / pause 비용 증가
```

## Allocation이란

객체, 배열, 문자열 등 새로운 managed object를 만드는 것이다.

```csharp
var user = new User();
var list = new List<int>();
var buffer = new byte[4096];
```

Allocation 자체는 .NET에서 매우 빠를 수 있다. 하지만 대량으로 반복되면 결국 GC가 처리해야 한다.

## Value Type과 Reference Type

C#의 대표적 Value Type:

- `int`
- `long`
- `bool`
- `double`
- `struct`

Reference Type:

- `class`
- `string`
- array
- delegate

하지만 "Value Type은 항상 Stack, Reference Type은 항상 Heap"이라고 외우면 틀릴 수 있다.

실제 저장 위치는 사용 문맥, JIT 최적화, 객체 내부 포함 여부 등에 따라 달라질 수 있다.

면접에서는 더 안전하게:

> Value Type은 값 자체를 보유하고 Reference Type은 객체에 대한 참조를 다룬다. 저장 위치를 단순히 Stack/Heap으로 1:1 대응시키면 안 된다.

라고 설명하는 편이 정확하다.

## Boxing

Value Type을 `object`나 특정 interface 형태로 다룰 때 값이 object 형태로 포장되는 것을 Boxing이라고 한다.

```csharp
int x = 42;
object o = x;
```

개념적으로:

```text
int value
→ object로 감싸기
→ managed allocation 가능
```

반대로 object에서 원래 Value Type을 꺼내는 것을 Unboxing이라고 한다.

```csharp
int y = (int)o;
```

## Boxing이 왜 문제인가?

한두 번은 별 문제 없다.

하지만 hot path에서 매우 자주 발생하면:

- allocation 증가
- GC pressure 증가
- copy/cast 비용

이 쌓일 수 있다.

## Generics가 중요한 이유

```csharp
ArrayList list = new ArrayList();
list.Add(1);
```

과거 non-generic collection에서는 Value Type이 object로 들어가며 Boxing이 발생하기 쉬웠다.

```csharp
List<int> list = new List<int>();
list.Add(1);
```

Generic을 쓰면 Value Type을 그대로 다룰 수 있어 불필요한 Boxing을 줄일 수 있다.

## String Allocation

`string`은 immutable이다.

```csharp
string s = "";
for (...)
{
    s += value;
}
```

반복 연결은 많은 중간 문자열을 만들 수 있다.

이럴 때는 상황에 따라 `StringBuilder`가 더 적절할 수 있다.

다만 작은 문자열 몇 개를 합치는 코드까지 무조건 StringBuilder로 바꾸는 것은 과최적화다.

## LINQ와 Allocation

LINQ는 생산성과 가독성을 높이지만 hot path에서는 iterator/delegate/closure 등과 관련된 allocation이 생길 수 있다.

중요한 원칙:

> LINQ가 느리다고 외우지 말고 profiler로 실제 hot path인지 확인한다.

## Closure Capture

```csharp
int threshold = 10;
var result = items.Where(x => x > threshold);
```

lambda가 외부 변수를 capture하면 compiler가 closure object를 생성할 수 있다.

이 역시 일반 비즈니스 코드에서는 문제가 없을 수 있지만 초고빈도 경로에서는 allocation 원인이 될 수 있다.

## ArrayPool과 Buffer Reuse

큰 buffer를 반복 생성하는 경우:

```csharp
byte[] buffer = new byte[1024 * 1024];
```

매번 새 배열을 만들기보다 `ArrayPool<T>`로 buffer를 빌려 재사용할 수 있다.

```csharp
var buffer = ArrayPool<byte>.Shared.Rent(size);
try
{
    // use
}
finally
{
    ArrayPool<byte>.Shared.Return(buffer);
}
```

하지만 Pool 사용도 공짜는 아니다.

- Return 누락
- 민감 데이터 잔존
- 실제 필요한 크기보다 큰 배열 반환
- lifetime 관리 복잡성

을 고려해야 한다.

## Span<T> / Memory<T>

`Span<T>`는 기존 memory 영역의 일부를 새로운 배열 복사 없이 다루는 데 유용하다.

예:

```text
전체 byte[] 복사
vs
기존 buffer의 slice view
```

불필요한 allocation/copy를 줄이는 데 도움이 된다.

`Span<T>`는 stack-only 제약 등이 있으므로 async boundary를 넘을 때는 `Memory<T>`가 필요한 경우도 있다.

## TIFF→PDF 사례 연결

이미지 처리에서 매 Page마다:

- RGBA buffer 생성
- Bitmap 생성
- MemoryStream 생성
- 중간 byte[] 복사

가 반복되면 allocation rate가 매우 커질 수 있다.

성능 개선 방향은 단순히 CPU parallelism이 아니라:

- buffer reuse
- 불필요한 중간 copy 제거
- lifetime 단축
- 한꺼번에 처리하는 Page 수 제한

도 포함된다.

## Premature Optimization

모든 `new`를 없애는 것이 목표가 아니다.

먼저 확인할 것:

1. allocation rate가 실제로 높은가?
2. 어느 Type이 많이 생성되는가?
3. GC pause/CPU에 영향이 있는가?
4. hot path인가?

도구 예:

- `dotnet-counters`
- `dotnet-trace`
- profiler
- allocation profile

## 흔한 오해

### "new는 느리니까 쓰면 안 된다"

아니다. .NET allocation 자체는 빠른 경우가 많다. 문제는 누적 allocation과 GC cost다.

### "struct면 무조건 Heap allocation이 없다"

아니다. 문맥과 boxing 등에 따라 allocation이 생길 수 있다.

### "LINQ는 항상 느리다"

아니다. 실제 workload와 hot path를 측정해야 한다.

## 60초 면접 답변

> .NET에서 allocation 자체는 빠른 편이지만 Backend hot path에서 temporary object를 대량 생성하면 allocation rate가 올라가 GC 빈도와 CPU 비용이 증가할 수 있습니다. Boxing은 int 같은 Value Type을 object나 interface 형태로 포장하면서 allocation이 생길 수 있는 대표 사례이고, generics를 사용하면 불필요한 boxing을 줄일 수 있습니다. 또 문자열 반복 연결, closure, LINQ, 큰 byte array 같은 패턴도 workload에 따라 allocation source가 될 수 있습니다. 하지만 모든 new를 제거하는 식으로 최적화하기보다 profiler로 allocation rate와 hot type을 측정한 뒤 ArrayPool, Span, buffer reuse 같은 방법을 필요한 경로에 적용하는 것이 중요합니다.
