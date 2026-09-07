# 12. Async Stream / Channel

## 한 줄 요약

`IAsyncEnumerable<T>`는 **데이터를 한꺼번에 다 만들지 않고 준비되는 대로 비동기적으로 흘려보내는 방식**이고, `Channel<T>`은 Producer와 Consumer 사이를 연결하는 **비동기 Queue + Backpressure 도구**다.

## 왜 필요한가

백엔드에서 결과 10만 건을 모두 메모리에 만든 뒤 반환하면 latency와 memory가 커진다. 반대로 한 건씩 준비될 때마다 흘려보내면 첫 응답을 빨리 시작하고 peak memory를 줄일 수 있다.

```csharp
await foreach (var item in GetItemsAsync(ct))
{
    await ProcessAsync(item, ct);
}
```

`IAsyncEnumerable<T>`는 각 항목이 준비될 때까지 비동기 대기할 수 있다.

## Async Stream != Parallelism

비동기 스트림은 데이터를 점진적으로 소비하는 abstraction이다. 자동으로 여러 항목을 병렬 처리하는 것은 아니다.

## Channel<T>

Producer와 Consumer 속도가 다를 때 queue가 필요하다.

```text
Producer -> Channel -> Consumer
```

`Channel<T>`은 async read/write를 지원하고 bounded channel을 사용하면 queue capacity를 제한할 수 있다.

## Bounded Channel과 Backpressure

무한 queue는 문제를 미룰 뿐이다. Producer가 계속 더 빠르면 memory와 latency가 계속 증가한다.

Bounded Channel은 capacity가 찼을 때:
- producer를 기다리게 하거나
- 일부 item을 drop하거나
- 별도 정책을 적용할 수 있다.

즉 runtime 수준에서 backpressure를 구현할 수 있다.

## 언제 유용한가

- background job pipeline
- file/image processing pipeline
- log/event processing
- streaming API
- batch ETL

예를 들어 TIFF 변환 작업도 입력 생성 → 변환 → 저장 단계를 channel로 분리하면 각 단계의 concurrency를 제한할 수 있다. 다만 CPU-bound 변환을 무작정 병렬화하면 오히려 contention이 생길 수 있다.

## Cancellation과 Completion

장수명 stream/channel은 종료 조건이 중요하다.

- CancellationToken 전파
- writer completion
- reader 종료
- exception propagation

이 중 하나라도 빠지면 background task가 남거나 shutdown이 지연될 수 있다.

## 흔한 오해

- async stream은 자동 병렬 처리다 → 아니다.
- Channel은 Kafka 같은 durable message broker다 → 아니다. process memory 기반 coordination 도구다.
- unbounded channel이 빠르니 항상 좋다 → 아니다. overload 때 위험하다.

## 60초 면접 답변

`IAsyncEnumerable<T>`는 전체 결과를 먼저 materialize하지 않고 항목이 준비되는 대로 비동기적으로 전달하는 스트리밍 abstraction입니다. `Channel<T>`은 producer와 consumer 사이를 연결하는 비동기 queue로, bounded capacity를 사용하면 backpressure도 걸 수 있습니다. 둘 다 Thread를 자동으로 늘리거나 병렬 처리하는 기능은 아닙니다. Backend에서는 streaming response, background pipeline, batch 처리 등에 유용하고, cancellation, completion, exception propagation을 함께 설계해야 장수명 task가 남지 않습니다.