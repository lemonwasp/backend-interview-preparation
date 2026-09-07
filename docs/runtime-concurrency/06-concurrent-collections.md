# 06. Concurrent Collections

## 한 줄 요약

여러 Thread가 동시에 접근할 수 있는 자료구조라고 해서 내부 경쟁 비용이 사라지는 것은 아니다. Concurrent Collection은 **동시 접근을 안전하게 만들기 위한 도구**이지, 모든 복합 연산을 자동으로 Atomic하게 만들어주는 마법은 아니다.

## 파인만식 설명

일반 `Dictionary`를 여러 사람이 동시에 수정하는 공용 화이트보드라고 생각해보자.

한 사람이 쓰는 동안 다른 사람이 지우거나 같은 위치에 쓰면 상태가 깨질 수 있다.

`ConcurrentDictionary`는 여러 사람이 동시에 작업해도 내부 구조가 망가지지 않도록 동시성 제어를 제공한다. 하지만 다음은 여전히 조심해야 한다.

```csharp
if (!dict.ContainsKey(key))
{
    dict[key] = CreateValue();
}
```

`ContainsKey`와 대입은 각각 Thread-safe일 수 있어도 **두 연산 전체가 하나의 Atomic Operation인 것은 아니다.**

이런 경우에는 `GetOrAdd`처럼 복합 연산을 원자적으로 표현하는 API를 사용해야 한다.

## 대표 .NET Concurrent Collections

- `ConcurrentDictionary<TKey, TValue>`
- `ConcurrentQueue<T>`
- `ConcurrentStack<T>`
- `ConcurrentBag<T>`
- `Channel<T>`는 전통적 Collection과는 조금 다르지만 Producer/Consumer 모델에서 매우 중요하다.

## ConcurrentDictionary

### 잘못된 패턴

```csharp
if (!cache.ContainsKey(key))
{
    cache[key] = Load();
}
```

두 Thread가 동시에 `ContainsKey == false`를 보고 둘 다 `Load()`를 실행할 수 있다.

### 더 나은 표현

```csharp
var value = cache.GetOrAdd(key, _ => Load());
```

다만 중요한 함정이 있다.

`GetOrAdd`의 value factory는 경쟁 상황에서 **한 번만 실행된다고 보장되는 것이 아니다.** 최종적으로 Dictionary에 들어가는 값은 하나지만, factory 자체는 여러 번 실행될 수 있다.

따라서 factory 안에 결제, 외부 API 호출 같은 중복 불가 Side Effect를 넣으면 안 된다.

## Queue와 Producer/Consumer

`ConcurrentQueue<T>`는 동시 Enqueue/Dequeue에 안전하지만, 무한히 쌓이도록 두면 메모리 문제가 생길 수 있다.

실무에서는 `Channel<T>` 같은 bounded queue를 사용해:

- 최대 queue 길이 제한
- producer backpressure
- async read/write
- cancellation

을 함께 처리하는 경우가 많다.

## Thread-safe와 Atomic은 다르다

Thread-safe Collection은 내부 자료구조가 깨지지 않도록 보호한다.

하지만 비즈니스 로직 전체가 Atomic하다는 뜻은 아니다.

예:

```text
재고 조회
→ 재고 1 감소
→ 주문 생성
```

이 전체가 Atomic해야 한다면 `ConcurrentDictionary`만으로 해결할 수 없다. DB Transaction, Lock, Compare-and-Swap 등 더 큰 경계가 필요하다.

## Lock-free라는 말도 조심

일부 Concurrent Collection은 Lock-free 또는 fine-grained locking 기법을 사용할 수 있다. 하지만 인터뷰에서 내부 구현을 일반화하면 위험하다.

정확한 표현:

> Concurrent Collection은 보통 일반 Collection에 외부 `lock`을 두는 것보다 동시성에 최적화된 구현을 제공하지만, 구현 세부는 자료구조와 Runtime 버전에 따라 다를 수 있다.

## Backend 연결

### 캐시

```csharp
ConcurrentDictionary<string, UserProfile>
```

같은 in-process cache를 만들 수 있다.

하지만 multi-instance 환경에서는 Instance A와 B의 Dictionary가 서로 공유되지 않는다.

즉:

```text
Thread-safe != Distributed-safe
```

### 작업 Queue

여러 Producer가 작업을 넣고 여러 Consumer가 처리할 때 ConcurrentQueue/Channel이 유용하다.

### Rate Limiter

카운터를 ConcurrentDictionary에 넣었다고 해서 여러 서버 Instance 간 Rate Limit이 일관되는 것은 아니다.

분산 환경이라면 Redis, DB, centralized limiter 같은 별도 coordination이 필요하다.

## 흔한 오해

### "ConcurrentDictionary면 Lock이 필요 없다"

항상 그렇지 않다. Collection 단일 연산은 안전해도 여러 연산을 하나의 논리적 원자성으로 묶어야 하면 추가 동기화가 필요할 수 있다.

### "GetOrAdd factory는 딱 한 번 실행된다"

아니다. 경쟁 시 여러 번 실행될 수 있다.

### "Thread-safe면 Process-safe다"

아니다. 같은 Process 내부 Thread 동시성만 다루는 경우가 대부분이다.

## 60초 면접 답변

> Concurrent Collection은 여러 Thread가 동시에 접근해도 내부 자료구조가 깨지지 않도록 설계된 Collection입니다. .NET에서는 ConcurrentDictionary, ConcurrentQueue 등이 대표적입니다. 다만 Thread-safe가 비즈니스 연산 전체의 Atomicity를 뜻하지는 않습니다. 예를 들어 ContainsKey 후 Add처럼 여러 연산을 조합하면 Race Condition이 생길 수 있어 GetOrAdd 같은 Atomic API를 사용해야 합니다. 또한 GetOrAdd의 factory는 경쟁 상황에서 여러 번 실행될 수 있으므로 Side Effect를 넣으면 안 됩니다. 그리고 Concurrent Collection은 Process 내부 동시성 도구이므로 여러 서버 Instance 사이의 일관성 문제는 Redis나 DB 같은 별도 coordination이 필요합니다.
