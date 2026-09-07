# 18. Socket Internals

## 이번 학습 목표

- Socket을 단순한 API가 아니라 Kernel 네트워크 객체로 이해한다.
- `socket → bind → listen → accept` 흐름을 설명한다.
- Send Buffer와 Receive Buffer의 역할을 이해한다.
- Blocking / Non-blocking Socket의 차이를 OS 관점에서 설명한다.
- Connection 하나가 어떤 Kernel 자원을 소비하는지 이해한다.

---

## 1. Socket은 네트워크용 File Descriptor다

Unix 계열에서 Socket을 생성하면 애플리케이션은 FD를 받습니다.

```text
socket()
  ↓
Kernel socket object 생성
  ↓
FD 반환
```

애플리케이션은 이 FD를 통해 Kernel network stack과 통신합니다.

---

## 2. 서버 Socket의 기본 흐름

```text
socket()
  ↓
bind()
  ↓
listen()
  ↓
accept()
  ↓
client connection용 새 socket FD
```

중요한 점은 listening socket과 accepted connection socket이 다르다는 것입니다.

- listening socket: 새 연결을 받는 입구
- connected socket: 특정 client와 실제 데이터를 주고받는 통로

---

## 3. Send Buffer와 Receive Buffer

Socket에는 Kernel이 관리하는 buffer가 존재합니다.

```text
Application
   ↓ send
Kernel Send Buffer
   ↓ TCP/IP stack
NIC
```

수신은 반대입니다.

```text
NIC
  ↓
Kernel Receive Buffer
  ↓ recv/read
Application
```

애플리케이션의 `send()` 성공은 상대방 애플리케이션이 데이터를 읽었다는 뜻이 아닙니다.
대개 Kernel send buffer가 데이터를 받아들였다는 의미에 가깝습니다.

---

## 4. Buffer가 가득 차면?

Send Buffer가 가득 찬 상태에서 Blocking Socket에 추가로 쓰려고 하면
Thread가 기다릴 수 있습니다.

Non-blocking Socket에서는 즉시 성공하지 못하고
“지금은 쓸 수 없다”는 결과를 반환할 수 있습니다.

이것이 I/O Multiplexing과 연결됩니다.

---

## 5. accept queue

서버가 `listen()` 상태일 때 들어오는 연결은 Kernel 내부 queue에 관리됩니다.

서버가 `accept()`를 충분히 빠르게 처리하지 못하거나 부하가 너무 높으면
queue가 가득 차 연결 지연이나 실패가 발생할 수 있습니다.

실제 TCP 연결 queue는 구현상 더 세부적인 구조를 가지지만,
면접에서는 “Kernel이 pending connection을 queue로 관리한다”는 큰 그림을 먼저 이해하면 됩니다.

---

## 6. Connection 하나의 비용

TCP Connection 하나는 단순 숫자 하나가 아닙니다.

대표적으로 다음 자원을 소비합니다.

- FD
- Kernel socket object
- Send/Receive buffer
- TCP state
- timer와 retransmission 관련 상태
- 애플리케이션 측 connection object

따라서 수만 개 connection을 처리할 때는 Thread 수뿐 아니라 Kernel memory와 FD limit도 중요합니다.

---

## 7. Thread-per-connection의 문제

Connection마다 Thread를 하나씩 붙이면 다음 비용이 증가할 수 있습니다.

- Thread stack memory
- Context Switching
- Scheduler overhead

반면 non-blocking I/O + multiplexing 모델은 상대적으로 적은 Thread로 많은 Socket 상태를 관리할 수 있습니다.

---

## 8. Socket과 async/await

C#에서 `await socket.ReceiveAsync(...)` 같은 코드를 사용한다고 해서
connection마다 전용 Thread가 기다리는 것은 아닙니다.

OS의 비동기/이벤트 기반 네트워크 메커니즘과 Runtime이 결합해
I/O 완료 시 continuation을 실행할 수 있습니다.

이 때문에 많은 동시 연결을 Thread 수와 1:1로 매핑하지 않고 처리할 수 있습니다.

---

## 9. 60초 면접 답변

> Socket은 애플리케이션이 Kernel network stack을 사용하기 위한 네트워크 통신 endpoint입니다. Unix 계열에서는 File Descriptor로 표현됩니다. 서버는 `socket`, `bind`, `listen`, `accept` 흐름으로 연결을 받고, accepted connection마다 별도의 connected socket이 만들어집니다. Kernel은 Socket별 Send/Receive buffer와 TCP state를 관리합니다. `send()`가 성공했다고 상대 애플리케이션이 읽었다는 뜻은 아니고, 보통 로컬 Kernel buffer가 데이터를 받아들였다는 의미입니다. 많은 connection을 처리할 때는 FD, Kernel buffer, TCP state, Thread model까지 함께 고려해야 합니다.

---

## 핵심 요약

```text
socket = Kernel network endpoint
listen socket ≠ connected socket
send/receive buffer = Kernel-managed
connection cost = FD + buffers + TCP state + app state
```

다음 주제: TCP가 OS에서 처리되는 과정
