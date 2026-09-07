# 15. Runtime Observability

## 한 줄 요약

Runtime Observability는 **CPU, ThreadPool, GC, allocation, exception, lock, request trace를 함께 관찰해 애플리케이션 내부 병목의 원인을 찾는 능력**이다.

## 왜 필요한가

"서버가 느립니다"만으로는 원인을 알 수 없다.

같은 latency 증가라도 원인은 다를 수 있다.

- CPU saturation
- ThreadPool starvation
- GC pause
- allocation 폭증
- lock contention
- DB/HTTP connection pool wait
- downstream latency

그래서 metric 하나가 아니라 여러 신호를 함께 봐야 한다.

## Metrics / Logs / Traces

### Metrics
시스템 상태의 숫자 추세를 본다.

예:
- request rate / latency / error rate
- CPU / memory
- GC collection count / pause
- allocation rate
- ThreadPool queue / thread count
- lock contention

### Logs
개별 사건의 상세 문맥을 남긴다.

좋은 log는 correlation/trace id를 포함해 요청 흐름과 연결되어야 한다.

### Traces
한 요청이 서비스, DB, 외부 API를 통과하는 시간을 span 단위로 분해한다.

```text
HTTP request 850ms
  App code       20ms
  DB query      120ms
  External API  680ms
```

이렇게 보면 "앱이 느리다"가 아니라 실제 병목 위치를 찾을 수 있다.

## .NET Runtime에서 자주 볼 것

- process CPU
- working set / managed heap
- allocation rate
- Gen 0/1/2 collection
- GC pause
- LOH
- ThreadPool queue length
- ThreadPool thread count
- exception rate
- monitor lock contention

## 대표 도구

- `dotnet-counters`: live runtime counters
- `dotnet-trace`: EventPipe trace 수집
- `dotnet-dump`: crash/hang/memory dump 분석
- `dotnet-gcdump`: GC heap 분석
- OpenTelemetry / APM: metrics, logs, distributed traces

## 진단 예시 1: latency 상승 + CPU 낮음

ThreadPool queue가 증가하고 worker 수가 늘고 있다면 starvation을 의심한다.

다음으로 blocking stack, `.Result`, `.Wait()`, lock wait를 찾는다.

## 진단 예시 2: memory 상승

Managed heap size도 함께 증가하는가?

- 증가한다 → retained object / unbounded cache / event subscription 등을 의심
- managed heap은 안정적이나 RSS만 증가 → native buffer, fragmentation, memory mapping 등도 고려

## 진단 예시 3: p99만 튄다

평균만 보면 놓칠 수 있다.

- GC pause
- occasional lock contention
- slow downstream
- queueing

같은 tail latency 원인을 trace와 runtime event로 확인한다.

## RED / USE 감각

Service 관점:
- Rate
- Errors
- Duration

Resource 관점:
- Utilization
- Saturation
- Errors

이 두 프레임을 함께 쓰면 진단 질문을 구조화하기 쉽다.

## 측정 전 최적화 금지

Allocation을 줄이는 코드가 복잡성을 크게 늘릴 수 있다. 모든 boxing이나 allocation을 제거하는 것이 목표가 아니다.

먼저 profile하고 실제 hot path에서 의미 있는 비용인지 확인한다.

## 60초 면접 답변

`Runtime observability는 애플리케이션 내부 병목을 metrics, logs, traces와 runtime event를 함께 사용해 진단하는 것입니다. .NET에서는 CPU, managed heap, allocation rate, GC pause, ThreadPool queue와 thread count, exception, lock contention을 자주 봅니다. 예를 들어 CPU는 낮은데 latency와 ThreadPool queue가 증가하면 starvation을 의심하고 blocking stack을 추적합니다. memory가 증가할 때도 RSS만 볼 게 아니라 managed heap과 retention을 같이 봅니다. 핵심은 평균 하나가 아니라 p95/p99, queueing, downstream trace를 연결해 측정한 뒤 최적화하는 것입니다.