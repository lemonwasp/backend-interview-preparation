# 18. gRPC

## 학습 목표

- gRPC가 무엇인지 설명한다.
- Protocol Buffers와 HTTP/2의 역할을 설명한다.
- Unary / Server Streaming / Client Streaming / Bidirectional Streaming을 구분한다.
- REST API와 비교해 언제 유리한지 설명한다.
- 내부 서비스 통신에서 gRPC를 사용할 때의 trade-off를 말할 수 있다.

---

## 1. 가장 쉬운 설명

gRPC는 원격 서버의 함수를 마치 로컬 함수를 호출하듯 사용하도록 설계된 RPC framework입니다.

예를 들어 서비스 정의가 다음과 같다고 합시다.

```proto
service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}
```

Client에서는 생성된 Stub을 통해 호출합니다.

```text
client.GetUser(...)
```

하지만 실제로는 네트워크를 통해 다른 Process/Server로 요청이 전달됩니다.

즉,

> gRPC = strongly-typed service contract + generated client/server code + efficient RPC transport

로 이해할 수 있습니다.

---

## 2. Protocol Buffers

gRPC에서 흔히 사용하는 IDL과 serialization format이 Protocol Buffers(Protobuf)입니다.

```proto
message User {
  int64 id = 1;
  string name = 2;
}
```

장점:

- schema가 명확함
- binary format으로 compact함
- code generation 가능
- 여러 언어 간 contract 공유에 유리

단점:

- 사람이 직접 읽기 어렵다
- schema/version 관리가 필요하다
- JSON 기반 API보다 debugging 도구가 덜 직관적일 수 있다

---

## 3. HTTP/2를 사용하는 이유

gRPC는 일반적으로 HTTP/2 위에서 동작합니다.

HTTP/2가 제공하는 중요한 특성:

- 하나의 connection에서 여러 stream
- binary framing
- flow control
- header compression
- streaming 지원

이 덕분에 서비스 간 다수 RPC를 효율적으로 처리할 수 있습니다.

---

## 4. 네 가지 RPC 형태

### Unary

```text
1 Request → 1 Response
```

일반적인 함수 호출과 가장 비슷합니다.

### Server Streaming

```text
1 Request → N Responses
```

예: 대량 결과 stream

### Client Streaming

```text
N Requests → 1 Response
```

예: chunk upload

### Bidirectional Streaming

```text
N Requests ↔ N Responses
```

양쪽이 독립적으로 stream을 주고받습니다.

---

## 5. REST보다 무조건 빠른가?

아닙니다.

Protobuf와 HTTP/2 때문에 payload와 connection 사용 측면에서 효율적인 경우가 많지만 전체 latency는 다음에도 좌우됩니다.

- DB
- downstream API
- network RTT
- serialization 비용
- business logic

따라서 gRPC 선택 이유를 단순히 "빠르다"라고만 말하면 부족합니다.

더 좋은 설명은:

> 엄격한 contract, code generation, streaming, 내부 서비스 간 효율적인 통신이 필요한 환경에서 특히 유리하다.

입니다.

---

## 6. Deadline과 Cancellation

분산 시스템에서 RPC는 영원히 기다리면 안 됩니다.

Client는 deadline을 설정해야 합니다.

```text
Service A → Service B → Service C
```

B가 30초를 기다리면 A의 Thread/connection/resource도 묶일 수 있습니다.

gRPC는 deadline과 cancellation propagation을 지원합니다.

이 개념은 이후 Timeout / Retry / Circuit Breaker와 연결됩니다.

---

## 7. Error Handling

gRPC는 HTTP status code만 사용하는 것이 아니라 gRPC status code를 사용합니다.

예:

- OK
- INVALID_ARGUMENT
- NOT_FOUND
- ALREADY_EXISTS
- PERMISSION_DENIED
- UNAUTHENTICATED
- RESOURCE_EXHAUSTED
- UNAVAILABLE
- DEADLINE_EXCEEDED

Error code를 Retry 가능 여부와 연결해 설계하는 것이 중요합니다.

---

## 8. Versioning

Protobuf field number는 contract의 중요한 일부입니다.

```proto
string name = 2;
```

Field를 삭제했다고 같은 번호를 다른 의미로 재사용하면 compatibility 문제가 생길 수 있습니다.

안전한 진화 원칙:

- 기존 field number를 함부로 재사용하지 않는다
- additive change를 선호한다
- 구/신 Client가 공존할 수 있는지 생각한다

---

## 9. gRPC와 Browser

브라우저 환경에서는 native gRPC 사용에 제약이 있어 gRPC-Web이나 proxy 계층을 사용하는 경우가 있습니다.

따라서 public browser API에 무조건 gRPC가 가장 적합한 것은 아닙니다.

---

## 10. Backend 사례

Microservice 내부:

```text
API Gateway
    │ REST/HTTP
    ▼
Backend-for-Frontend
    │ gRPC
    ├── User Service
    ├── Order Service
    └── Payment Service
```

외부에는 HTTP/JSON을 제공하고 내부는 gRPC를 사용할 수도 있습니다.

---

## 11. 자주 하는 오해

### gRPC = Protobuf

완전히 같은 개념은 아닙니다. Protobuf는 흔히 사용하는 IDL/serialization format이고 gRPC는 RPC framework입니다.

### gRPC = HTTP/2

gRPC가 일반적으로 HTTP/2를 transport로 사용하지만 둘은 같은 계층의 개념이 아닙니다.

### gRPC면 Network Failure가 사라진다

아닙니다. 원격 호출은 여전히 timeout, retry, partial failure를 고려해야 합니다.

---

## 12. 60초 면접 답변

> gRPC는 Protocol Buffers로 service contract를 정의하고 client/server stub을 생성해 원격 서비스를 strongly-typed 함수 호출처럼 사용할 수 있게 하는 RPC framework입니다. 일반적으로 HTTP/2를 사용해서 하나의 connection에서 여러 stream과 unary, server streaming, client streaming, bidirectional streaming을 지원합니다. 내부 microservice 통신에서 contract 일관성, code generation, compact binary payload가 장점이지만 public browser API에서는 tooling과 compatibility 때문에 REST가 더 편할 수 있습니다. 또한 원격 호출이므로 deadline, cancellation, retry 가능 error와 schema versioning을 반드시 고려해야 합니다.
