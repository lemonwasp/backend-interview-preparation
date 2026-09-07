# 11. TLS와 HTTPS

## 학습 목표

- HTTPS가 HTTP에 무엇을 추가하는지 설명할 수 있다.
- TLS가 기밀성, 무결성, 서버 인증을 어떻게 제공하는지 설명할 수 있다.
- TLS Handshake와 Application Data 전송을 구분할 수 있다.
- 대칭키와 비대칭키의 역할을 구분할 수 있다.
- TLS가 네트워크 지연과 CPU 비용에 어떤 영향을 주는지 설명할 수 있다.

---

## 1. HTTPS는 무엇인가?

HTTPS는 간단히 말해 **HTTP를 TLS 위에서 사용하는 것**입니다.

```text
Application: HTTP
Security:    TLS
Transport:   TCP  (HTTP/1.1, HTTP/2)
             QUIC (HTTP/3)
```

HTTP 자체는 메시지 형식과 의미를 정의하지만, 평문 HTTP만으로는 중간 네트워크가 내용을 볼 수 있고 변조할 수도 있습니다.

TLS는 여기에 세 가지 중요한 성질을 추가합니다.

1. **기밀성**: 제3자가 내용을 쉽게 읽지 못하게 암호화
2. **무결성**: 전송 중 데이터가 바뀌었는지 탐지
3. **인증**: 통신 상대가 기대한 서버인지 확인

---

## 2. 왜 대칭키와 비대칭키를 함께 쓸까?

비대칭 암호는 공개키와 개인키를 사용하며 인증과 키 합의에 유리하지만, 대용량 데이터를 계속 처리하기에는 상대적으로 비용이 큽니다.

대칭 암호는 같은 비밀키를 공유하고 매우 빠르게 데이터를 암복호화할 수 있습니다.

그래서 TLS는 개념적으로 다음처럼 동작합니다.

```text
Handshake
  ↓
서버 신원 확인 + 안전한 키 합의
  ↓
세션용 대칭키 생성
  ↓
Application Data는 대칭키로 빠르게 암호화
```

현대 TLS에서는 단순히 서버 공개키로 세션키를 암호화하는 방식보다 ECDHE 같은 키 합의 방식이 일반적입니다.

---

## 3. TLS 1.3 Handshake를 단순화하면

```text
Client
  | ClientHello
  | - supported versions
  | - cipher suites
  | - key share
  v
Server
  | ServerHello
  | Certificate
  | CertificateVerify
  | Finished
  v
Client
  | Certificate 검증
  | Finished
  v
Encrypted Application Data
```

클라이언트는 서버가 보낸 Certificate Chain을 검증하고, 해당 인증서가 접속하려는 Domain에 유효한지도 확인합니다.

Handshake가 끝난 뒤 HTTP Request와 Response는 암호화된 TLS Record 형태로 전달됩니다.

---

## 4. TLS는 패킷을 숨기는가?

아닙니다.

TLS는 주로 Application Payload를 보호합니다. 네트워크 전달을 위해 필요한 IP 주소, 패킷 크기, 타이밍 같은 일부 Metadata는 여전히 관찰될 수 있습니다.

따라서:

> HTTPS = 인터넷에서 아무 정보도 관찰할 수 없게 만드는 기술

은 잘못된 설명입니다.

---

## 5. TLS 비용

TLS에는 비용이 있습니다.

- Handshake RTT
- 인증서 검증
- 키 교환 연산
- 암호화 / 복호화 CPU 비용
- 추가 Record 처리

하지만 현대 CPU의 암호화 지원, Connection Reuse, Session Resumption 등으로 많은 비용을 줄일 수 있습니다.

백엔드에서는 새 연결을 매 Request마다 만드는 것보다 Connection Pool과 Keep-Alive를 사용하는 이유가 TCP뿐 아니라 TLS Handshake 비용과도 연결됩니다.

---

## 6. TLS Termination

대규모 서비스에서는 Application Server가 직접 TLS를 종료하지 않고 Load Balancer나 Reverse Proxy가 TLS를 처리하는 경우가 많습니다.

```text
Client
  ↓ HTTPS
Load Balancer / Reverse Proxy
  ↓ HTTP 또는 HTTPS
Backend
```

이를 TLS Termination이라고 부릅니다.

장점:

- 인증서 관리 중앙화
- 암호화 부하 집중 처리
- Backend 단순화

단점:

- Proxy 이후 구간을 평문으로 두면 내부 네트워크 보호가 약해질 수 있음
- End-to-end encryption 요구와 충돌 가능

---

## 7. 백엔드 면접에서 연결할 포인트

- HTTPS = HTTP + TLS
- TLS는 암호화만이 아니라 인증과 무결성도 제공
- Handshake에서는 비대칭키 기반 인증/키 합의, 이후 데이터는 대칭키 중심
- Connection Reuse는 TCP + TLS Handshake 비용을 줄임
- TLS Termination 위치는 시스템 설계 문제

---

## 60초 면접 답변

> HTTPS는 HTTP를 TLS 위에서 전송하는 방식입니다. TLS는 기밀성, 무결성, 서버 인증을 제공합니다. 연결을 시작할 때 TLS Handshake에서 서버 인증서를 검증하고 안전하게 세션 키를 합의한 뒤, 실제 HTTP 데이터는 성능이 좋은 대칭키 암호로 보호합니다. TLS에는 Handshake RTT와 암호화 비용이 있으므로 Connection Reuse와 Session Resumption이 중요합니다. 대규모 시스템에서는 Load Balancer나 Reverse Proxy에서 TLS를 종료하기도 합니다.

---

## 핵심 요약

```text
HTTPS = HTTP + TLS
TLS = Confidentiality + Integrity + Authentication
Handshake = 인증 + 키 합의
Application Data = 주로 대칭키 암호
```

다음 주제: Certificate와 PKI
