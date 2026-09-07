# 17. Server-Sent Events (SSE)

## 학습 목표

- SSE가 무엇인지 설명한다.
- WebSocket과 차이를 설명한다.
- HTTP 기반 단방향 stream이라는 점을 이해한다.
- 자동 재연결과 EventSource의 장단점을 설명한다.

---

## 1. 가장 쉬운 설명

SSE는 Server가 Client에게 이벤트를 계속 밀어주는 방식입니다.

핵심은:

> Server → Client 단방향 실시간 스트림

입니다.

브라우저는 `EventSource` API를 사용해 연결을 열고, 서버는 `text/event-stream` 형식으로 데이터를 계속 보냅니다.

```text
Client ── HTTP connection ── Server
          ← event
          ← event
          ← event
```

---

## 2. 왜 필요한가?

알림, 주가, 로그 tail, 진행률, AI 응답 streaming처럼 Client가 Server로 실시간 데이터를 자주 보낼 필요가 없고 Server가 지속적으로 업데이트를 보내면 되는 경우가 많습니다.

이때 WebSocket보다 SSE가 더 단순할 수 있습니다.

---

## 3. HTTP 기반이라는 점

SSE는 별도 양방향 protocol로 전환하는 것이 아니라 HTTP response body를 오래 열어둔 채 event를 계속 전송합니다.

대표 Content-Type:

```text
Content-Type: text/event-stream
```

Event 형식 예:

```text
event: progress
data: 70

```

빈 줄이 한 event의 끝을 나타냅니다.

---

## 4. 자동 재연결

브라우저 `EventSource`는 연결이 끊기면 자동 재연결을 지원합니다.

Server는 event에 `id`를 붙일 수 있고, Client는 재연결 시 `Last-Event-ID`를 이용해 이어받기를 구현할 수 있습니다.

단, 정확히 한 번 전달이 자동 보장된다는 뜻은 아닙니다.

중복이나 누락 처리는 application 설계가 필요할 수 있습니다.

---

## 5. SSE vs WebSocket

| 항목 | SSE | WebSocket |
|---|---|---|
| 방향 | Server → Client | 양방향 |
| 기반 | HTTP stream | WebSocket protocol |
| Browser API | EventSource | WebSocket |
| 자동 재연결 | 기본 지원 | 직접 구현하는 경우가 많음 |
| 데이터 | 기본 text | text/binary |
| 적합 | 알림, progress, stream | 채팅, 게임, 협업 |

---

## 6. SSE도 장기 연결이다

SSE는 단순해 보여도 connection을 오래 유지합니다.

따라서 다음을 고려해야 합니다.

- connection 수
- proxy/load balancer timeout
- buffering
- keepalive
- disconnect 감지
- 서버 메모리

특히 중간 Proxy가 response를 buffering하면 event가 즉시 전달되지 않을 수 있습니다.

---

## 7. HTTP/1.1과 HTTP/2

HTTP/1.1에서는 브라우저의 origin당 connection 제한이 문제가 될 수 있습니다.

HTTP/2에서는 하나의 연결에서 여러 stream을 multiplex할 수 있으므로 이런 제약을 완화할 수 있습니다.

---

## 8. Backend 사례: AI Streaming

LLM 응답을 token 단위 또는 chunk 단위로 Client에게 계속 전달한다고 합시다.

Client가 중간에 서버로 양방향 실시간 데이터를 계속 보내지 않는다면 SSE는 구현이 단순합니다.

```text
Browser → POST request
Server  → SSE stream
        ← token chunk
        ← token chunk
        ← done
```

실제 API 설계에서는 POST와 SSE endpoint를 분리하거나 fetch streaming을 사용할 수도 있습니다.

---

## 9. 자주 하는 오해

### SSE는 polling이다

아닙니다. 연결을 유지하며 server가 event를 push합니다.

### SSE는 delivery guarantee를 제공한다

자동 재연결 기능은 있지만 exactly-once를 자동 제공하지는 않습니다.

### WebSocket이 항상 더 좋다

단방향 server push라면 SSE가 더 단순하고 HTTP infrastructure와 잘 맞을 수 있습니다.

---

## 10. 60초 면접 답변

> SSE는 HTTP 연결을 오래 유지하면서 Server가 Client에게 이벤트를 지속적으로 보내는 단방향 streaming 방식입니다. 브라우저에서는 EventSource를 사용할 수 있고 자동 재연결과 Last-Event-ID 같은 기능이 있어 알림, 진행률, 로그, AI 응답 스트리밍에 적합합니다. WebSocket은 양방향 통신이 필요한 채팅이나 협업에 더 적합하고, SSE는 Server Push만 필요할 때 구조가 더 단순합니다. 다만 SSE도 장기 연결이므로 proxy buffering, idle timeout, 연결 수와 재연결 시 중복 처리 등을 고려해야 합니다.
