# 13. ThreadPool Starvation Diagnostics

## 한 줄 요약

ThreadPool Starvation은 **일은 많은데 일을 처리할 worker가 충분히 돌아오지 못해 queue가 밀리는 상태**다. 특히 sync-over-async, blocking I/O, 긴 CPU 작업이 원인이 되기 쉽다.

## 증상

- 요청 latency 급증
- CPU는 100%가 아닌데 응답이 느림
- ThreadPool queue 증가
- active worker 증가가 늦음
- timeout과 5xx 증가
- DB/HTTP 자체는 빠른데 app 내부 대기 증가

## 왜 생기나

대표적으로:

```csharp
var result = SomeAsync().Result;
```

이 코드는 worker 하나를 기다리게 한다. 이런 요청이 많아지면 많은 worker가 대기 상태로 묶이고, continuation을 실행할 worker도 부족해진다.

또 다른 원인:
- blocking file/network call
- long lock wait
- CPU-heavy work를 request thread에서 수행
- 과도한 synchronous logging

## Starvation != CPU Saturation

CPU Saturation은 CPU가 계산으로 꽉 찬 상태다.
Starvation은 worker가 blocking에 묶여 **CPU가 남아도 일을 못 꺼내는 상태**일 수 있다.

## 관찰 지표

.NET에서는 다음 계열을 본다.

- ThreadPool thread count
- ThreadPool queue length
- completed work item rate
- process CPU
- request latency / throughput
- lock contention
- GC pause / allocation rate

도구 예:
- `dotnet-counters`
- `dotnet-trace`
- `dotnet-dump`
- APM tracing

## 진단 순서

1. latency가 어디서 늘었는지 trace로 분해
2. CPU saturation 여부 확인
3. ThreadPool queue/thread 증가 확인
4. blocking stack / sync-over-async 탐색
5. lock, DB pool, HTTP pool wait 확인
6. CPU-heavy work 여부 확인

## 개선

- async I/O는 end-to-end async 유지
- `.Result`, `.Wait()` 제거
- CPU-heavy 작업은 concurrency 제한 또는 worker 분리
- long lock 줄이기
- bounded queue/backpressure
- timeout/cancellation 적용

단순히 ThreadPool min thread 수만 늘리는 것은 증상 완화가 될 수 있지만 근본 원인이 blocking이라면 문제를 숨길 수 있다.

## 60초 면접 답변

`ThreadPool starvation은 요청량에 비해 worker가 부족한 상태인데, 단순 CPU 과부하와는 다릅니다. sync-over-async, blocking I/O, 긴 lock wait처럼 worker가 오래 반환되지 않을 때 CPU가 남아 있어도 queue가 밀릴 수 있습니다. 진단할 때는 ThreadPool queue length, thread count, completed work rate, CPU, request latency, lock/connection wait을 같이 봅니다. 개선은 end-to-end async, blocking 제거, CPU 작업 격리, bounded concurrency가 핵심이고 min thread를 올리는 것은 보조 수단입니다.