# 01. TCP/IP 계층

## 이번 학습 목표

- 네트워크 계층화를 왜 사용하는지 설명할 수 있다.
- Application / Transport / Internet / Link 계층의 역할을 구분할 수 있다.
- HTTP 요청 하나가 각 계층을 어떻게 통과하는지 설명할 수 있다.
- OSI 7계층과 TCP/IP 모델을 억지로 1:1 대응하지 않는다.

---

## 1. 왜 계층으로 나눌까?

네트워크 통신을 한 덩어리로 구현하면 애플리케이션 개발자가 NIC 제어, IP 경로 선택,
재전송, 연결 관리까지 모두 알아야 합니다. 계층화는 각 문제를 나눕니다.

```text
Application  : 무엇을 보낼 것인가?
Transport    : 프로세스 간 전달을 어떻게 신뢰성 있게 할 것인가?
Internet     : 목적지 호스트까지 어떻게 보낼 것인가?
Link         : 같은 네트워크 구간에서 실제 프레임을 어떻게 전달할 것인가?
```

핵심은 **각 계층이 아래 계층의 세부 구현을 감추고 위 계층에 더 단순한 인터페이스를 제공한다**는 점입니다.

---

## 2. TCP/IP 4계층

| 계층 | 대표 프로토콜 | 핵심 역할 |
|---|---|---|
| Application | HTTP, DNS, SMTP | 애플리케이션 간 메시지 의미 정의 |
| Transport | TCP, UDP | Process-to-Process 전달, Port, 신뢰성 |
| Internet | IP | Host-to-Host 전달, Routing |
| Link | Ethernet, Wi-Fi | 같은 링크에서 Frame 전달 |

### Application Layer

HTTP에서 Method, Header, Status Code 같은 의미를 정의합니다.

### Transport Layer

TCP는 Byte Stream, Sequence Number, ACK, 재전송, Flow/Congestion Control을 제공합니다.
UDP는 더 단순한 Datagram 전달을 제공합니다.

### Internet Layer

IP는 목적지 IP Address를 기준으로 Packet을 여러 Router를 거쳐 전달합니다.
기본적으로 신뢰성 보장은 하지 않습니다.

### Link Layer

현재 Hop에서 Frame을 실제 네트워크 장치로 전달합니다. Ethernet의 MAC Address가 대표적입니다.

---

## 3. Encapsulation

애플리케이션 데이터는 아래 계층으로 내려가며 각 계층의 Header가 붙습니다.

```text
HTTP Message
   ↓
TCP Segment
   ↓
IP Packet
   ↓
Ethernet Frame
```

수신 측에서는 반대로 Header를 해석하며 위 계층으로 올립니다.

```text
Frame → IP Packet → TCP Segment → HTTP Message
```

이 과정을 Encapsulation / Decapsulation이라고 합니다.

---

## 4. HTTP 요청 하나의 실제 흐름

브라우저가 서버에 요청한다고 가정합니다.

```text
Browser
  ↓ HTTP request
TCP
  ↓ segment
IP
  ↓ packet
Ethernet / Wi-Fi
  ↓ frame
NIC
  ↓
Network
```

서버에서는:

```text
NIC
 ↓
Link Layer
 ↓
IP
 ↓
TCP
 ↓
Socket Receive Buffer
 ↓
Web Server / Application
```

즉 HTTP만 안다고 네트워크 전체를 이해한 것은 아닙니다.

---

## 5. Port는 왜 필요한가?

IP Address는 Host를 찾고, Port는 그 Host 안에서 어떤 Application endpoint로 전달할지 구분합니다.

```text
203.0.113.10:443
      │       └─ Port
      └──────── IP
```

TCP 연결은 일반적으로 다음 정보 조합으로 구분할 수 있습니다.

```text
source IP
source port
destination IP
destination port
```

---

## 6. OSI 7계층과 TCP/IP

OSI 모델은 개념 학습에 유용하지만 실제 인터넷 구현을 설명할 때 TCP/IP 모델과 완전히 1:1로 맞추면 오히려 혼란이 생길 수 있습니다.

대략적으로는:

```text
OSI Application/Presentation/Session ≈ TCP/IP Application
OSI Transport                         ≈ TCP/IP Transport
OSI Network                           ≈ TCP/IP Internet
OSI Data Link/Physical                ≈ TCP/IP Link
```

정도로 이해하면 충분합니다.

---

## 7. 백엔드 개발과 연결

백엔드 장애를 계층별로 보면 원인을 좁히기 쉽습니다.

```text
HTTP 500        → Application 문제 가능성
Connection reset→ TCP 상태 / peer 종료 가능성
No route        → IP / Routing 문제 가능성
Packet loss     → Link / Network 경로 문제 가능성
```

실무에서는 한 계층만 보면 안 됩니다.

예를 들어 "API가 느리다"는 현상도 DNS, TCP handshake, TLS, HTTP, DB, Application CPU 모두 원인이 될 수 있습니다.

---

## 8. 60초 면접 답변

> TCP/IP 모델은 네트워크 통신을 Application, Transport, Internet, Link 계층으로 나누어 복잡성을 관리합니다. Application 계층은 HTTP 같은 메시지 의미를 정의하고, Transport 계층의 TCP나 UDP가 Process 간 전달을 담당합니다. Internet 계층의 IP는 목적지 Host까지 Packet을 Routing하고, Link 계층은 현재 네트워크 구간에서 Frame을 실제 장치로 전달합니다. 송신할 때는 각 계층 Header가 붙는 Encapsulation이 일어나고, 수신 측에서는 반대로 Decapsulation됩니다. 계층화 덕분에 애플리케이션은 NIC나 Routing의 세부 구현을 직접 알지 않아도 통신할 수 있습니다.

---

## 핵심 요약

```text
Application = 메시지 의미
Transport   = Process-to-Process
Internet    = Host-to-Host
Link        = Hop-to-Hop
```

다음 주제: TCP 3-way Handshake
