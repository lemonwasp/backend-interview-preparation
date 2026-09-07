# 19. TCP가 OS에서 처리되는 과정

## 이번 학습 목표

- TCP가 애플리케이션 라이브러리만으로 구현되는 것이 아님을 이해한다.
- Packet이 NIC에서 애플리케이션까지 오는 흐름을 설명한다.
- 애플리케이션의 `send()`가 NIC 전송으로 이어지는 흐름을 설명한다.
- TCP state, retransmission, ordering, flow control이 Kernel과 연결되는 방식을 이해한다.
- OS 파트와 네트워크 파트의 연결점을 잡는다.

---

## 1. TCP는 어디에서 동작할까?

일반적인 운영체제에서 TCP/IP stack의 핵심 처리는 Kernel에서 수행됩니다.

애플리케이션은 Socket API를 사용하지만,

```text
Application
   ↓ socket API
Kernel TCP/IP Stack
   ↓
NIC
```

실제 packet 생성, sequence number 관리, retransmission, receive ordering 같은 핵심 상태는 Kernel이 관리합니다.

---

## 2. Packet 수신 전체 흐름

단순화하면 다음과 같습니다.

```text
Network
   ↓
NIC
   ↓ DMA
RAM
   ↓ Interrupt / polling
Kernel network driver
   ↓
IP processing
   ↓
TCP processing
   ↓
Socket receive buffer
   ↓ recv/read
Application
```

각 계층을 순서대로 봅시다.

---

## 3. NIC와 DMA

NIC가 Ethernet frame을 받으면 packet data를 DMA를 통해 RAM의 buffer에 기록할 수 있습니다.

CPU가 모든 byte를 직접 복사하는 것이 아닙니다.

그 후 Interrupt나 polling 기반 메커니즘을 통해 Kernel이 packet arrival을 인식합니다.

---

## 4. Kernel network driver

Network driver는 NIC와 운영체제 사이의 연결 역할을 합니다.

Driver는 수신된 frame 정보를 Kernel network stack에 넘기고,
이후 Ethernet/IP/TCP 계층의 처리가 이어집니다.

---

## 5. IP 계층

IP 계층에서는 대표적으로 다음을 확인합니다.

- destination IP
- protocol
- routing 관련 정보
- packet validity

TCP packet이라면 TCP 처리 계층으로 전달됩니다.

---

## 6. TCP 계층

TCP는 단순히 payload만 전달하지 않습니다.

Kernel은 connection별로 다음 상태를 관리합니다.

- source/destination IP와 port
- sequence number
- acknowledgment number
- receive/send window
- retransmission timer
- connection state

예:

```text
SYN_SENT
SYN_RECEIVED
ESTABLISHED
FIN_WAIT
TIME_WAIT
```

---

## 7. 순서가 뒤섞여 도착하면?

TCP는 byte stream의 순서를 보장합니다.

Packet이 네트워크에서 순서가 바뀌어 도착할 수 있어도
Kernel TCP stack은 sequence number를 기반으로 데이터를 정렬하고
애플리케이션에는 순서 있는 byte stream을 제공합니다.

즉 애플리케이션은 보통 packet 단위가 아니라 stream을 읽습니다.

---

## 8. Packet이 손실되면?

TCP는 acknowledgment와 retransmission mechanism을 사용합니다.

필요한 ACK가 오지 않거나 손실이 감지되면 Kernel TCP stack이 재전송을 수행할 수 있습니다.

애플리케이션이 매 packet마다 직접 재전송 코드를 작성하는 것이 아닙니다.

---

## 9. Socket Receive Buffer

정상적으로 처리된 TCP payload는 해당 Socket의 Receive Buffer에 저장됩니다.

```text
TCP stack
  ↓
Socket receive buffer
  ↓
Application recv/read
```

애플리케이션이 느리게 읽으면 Receive Buffer가 차게 되고,
이 상태는 TCP flow control에도 영향을 줄 수 있습니다.

---

## 10. 송신 흐름

애플리케이션이 `send()`를 호출하면 반대 방향으로 진행됩니다.

```text
Application
  ↓ send
Socket send buffer
  ↓
TCP segmentation / sequence
  ↓
IP processing
  ↓
NIC driver
  ↓
NIC
  ↓
Network
```

Kernel은 데이터를 적절한 TCP segment로 나누고 header와 sequence 정보를 관리합니다.

---

## 11. send() 성공의 의미

`send()`가 성공했다고 해서 상대방이 데이터를 받았다는 뜻은 아닙니다.

대개 다음 정도를 의미합니다.

> 로컬 Kernel이 전송할 데이터를 받아들였다.

그 뒤에도 실제 network transmission, 상대방 수신, ACK 등이 남아 있을 수 있습니다.

---

## 12. Flow Control

TCP는 receiver가 감당할 수 있는 만큼만 보내도록 receive window를 사용합니다.

상대방 애플리케이션이 데이터를 느리게 읽으면
상대 Kernel의 receive buffer가 차고 advertised window가 줄어들 수 있습니다.

즉 애플리케이션 처리 속도가 TCP 전송 속도에 영향을 줄 수 있습니다.

---

## 13. Congestion Control

Flow Control은 receiver 보호가 목적이고,
Congestion Control은 network 혼잡을 제어하는 것이 목적입니다.

이 둘은 같은 개념이 아닙니다.

Kernel TCP stack은 congestion window 등을 사용해 네트워크 상황에 맞춰 전송량을 조절합니다.

구체적인 알고리즘은 다음 네트워크 파트에서 다룹니다.

---

## 14. Connection 상태와 TIME_WAIT

TCP 연결이 종료된 뒤에도 일정 시간 상태가 남을 수 있습니다.

TIME_WAIT는 불필요한 버그가 아니라
지연된 packet과 connection 재사용 문제를 안전하게 처리하기 위한 TCP 설계의 일부입니다.

대량의 짧은 connection을 만드는 서버에서는 TIME_WAIT가 관찰될 수 있습니다.

---

## 15. 백엔드 장애와 연결

다음 현상들은 TCP/OS 내부 상태와 연결됩니다.

- connection timeout
- connection reset
- socket buffer saturation
- accept queue saturation
- TIME_WAIT 증가
- retransmission 증가
- packet loss
- high network latency

따라서 단순히 애플리케이션 로그만 보는 것이 아니라
OS network metric도 함께 봐야 할 때가 있습니다.

---

## 16. 지금까지 배운 OS 개념 연결

```text
NIC
 ↓ DMA
Memory
 ↓ Interrupt / Polling
Kernel
 ↓ TCP/IP stack
Socket buffer
 ↓ File Descriptor
Application
```

여기에는 지금까지 배운 개념이 모두 들어갑니다.

- Kernel Mode
- System Call
- File Descriptor
- Interrupt
- DMA
- Socket
- Blocking / Non-blocking
- I/O Multiplexing
- Context Switching
- Buffer / Memory

---

## 17. 60초 면접 답변

> 일반적인 운영체제에서 TCP/IP stack의 핵심 처리는 Kernel이 담당합니다. Packet이 NIC에 도착하면 DMA를 통해 메모리 buffer에 기록되고 Interrupt나 polling을 통해 Kernel이 이를 처리합니다. Driver와 IP 계층을 거친 뒤 TCP stack이 sequence number, ACK, retransmission, flow control 같은 connection state를 처리하고 payload를 Socket receive buffer에 넣습니다. 애플리케이션은 `recv`나 `read`를 통해 이 데이터를 읽습니다. 송신은 반대로 애플리케이션이 Socket send buffer에 데이터를 넣으면 Kernel TCP stack이 segment와 sequence를 관리해 NIC로 보냅니다. 따라서 Socket API는 애플리케이션과 Kernel TCP stack 사이의 인터페이스라고 볼 수 있습니다.

---

## 핵심 요약

```text
TCP core logic = Kernel network stack
receive: NIC → DMA → Kernel → TCP → socket buffer → app
send: app → socket buffer → TCP → Kernel driver → NIC
TCP provides ordered reliable byte stream, not message boundaries
```

다음 파트: Networking Fundamentals
