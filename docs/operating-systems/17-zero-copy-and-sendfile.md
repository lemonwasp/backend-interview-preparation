# 17. Zero-copy와 sendfile

## 이번 학습 목표

- 일반 파일 전송에서 데이터가 어디를 복사되는지 설명한다.
- zero-copy가 무엇을 줄이려는 최적화인지 이해한다.
- `sendfile`의 기본 아이디어를 설명한다.
- zero-copy가 항상 완전한 0-copy를 뜻하지 않는다는 점을 이해한다.
- 웹 서버, 파일 서버, 프록시 성능과 연결한다.

---

## 1. 파일을 Socket으로 보내는 단순한 방법

애플리케이션이 파일을 읽어 네트워크로 전송한다고 생각해봅시다.

```text
Disk / Page Cache
      ↓
Kernel buffer
      ↓ copy
User-space buffer
      ↓ write/send
Kernel socket buffer
      ↓
NIC
```

고수준 코드에서는 단순히

```text
read file
send socket
```

처럼 보이지만, 내부에서는 User Space와 Kernel Space 사이의 데이터 이동이 발생할 수 있습니다.

---

## 2. 왜 복사가 비용인가?

큰 파일을 전송할수록 다음 비용이 커질 수 있습니다.

- 메모리 bandwidth 소비
- CPU가 데이터 복사에 사용됨
- cache pollution
- System Call과 경계 통과

애플리케이션이 파일 내용을 실제로 수정하거나 해석하지 않는다면
User-space buffer를 거치는 과정이 불필요할 수 있습니다.

---

## 3. zero-copy의 핵심 아이디어

> 애플리케이션이 데이터를 직접 가공하지 않는다면 불필요한 메모리 복사를 줄이자.

예를 들어 Kernel이 Page Cache의 파일 데이터를 Socket 전송 경로로 직접 연결할 수 있다면
User Space로 복사했다가 다시 Kernel로 보내는 단계를 줄일 수 있습니다.

```text
Page Cache
    ↓
Kernel networking path
    ↓
NIC
```

---

## 4. sendfile

Unix 계열의 `sendfile`은 파일 Descriptor와 Socket Descriptor를 Kernel에 전달해
파일 데이터를 User-space buffer를 거치지 않고 전송하도록 지원하는 대표적인 API입니다.

개념적으로:

```text
sendfile(socket_fd, file_fd, ...)
```

애플리케이션은 “이 파일의 이 범위를 이 Socket으로 보내라”고 Kernel에 요청합니다.

---

## 5. 정말 복사가 0번인가?

`zero-copy`라는 이름을 문자 그대로 받아들이면 안 됩니다.

실제 하드웨어와 OS 구현에 따라:

- DMA
- Page Cache
- NIC buffer
- descriptor 전달
- memory mapping

등이 사용되며 어떤 형태의 데이터 이동은 존재합니다.

핵심은 보통 **불필요한 CPU-mediated copy, 특히 User Space↔Kernel Space 복사를 줄이는 것**입니다.

---

## 6. 일반 read/write와 비교

### 전통적 방식

```text
file
 ↓
Kernel
 ↓ copy
User buffer
 ↓ copy
Kernel socket buffer
 ↓
NIC
```

### zero-copy 계열

```text
file/Page Cache
 ↓
Kernel networking path
 ↓
NIC
```

중간의 User-space 데이터 복사와 일부 System Call overhead를 줄일 수 있습니다.

---

## 7. 언제 유리한가?

특히 다음처럼 데이터 자체를 거의 가공하지 않고 전달할 때 유리합니다.

- 정적 파일 서버
- CDN
- 대용량 파일 다운로드
- reverse proxy 일부 경로
- 로그/데이터 전달 파이프라인

반대로 압축, 암호화, 변환처럼 애플리케이션이 내용을 적극 가공해야 하면 단순한 `sendfile` 경로를 그대로 사용하기 어려울 수 있습니다.

---

## 8. TLS와 zero-copy

HTTPS에서는 데이터가 암호화되어야 하므로 전송 경로가 복잡해집니다.

전통적으로는 TLS 처리를 User Space에서 수행해 단순 `sendfile`의 이점이 제한될 수 있습니다.
현대 OS와 NIC에는 Kernel TLS, hardware offload 등 다양한 최적화가 있지만

> HTTPS = 항상 zero-copy 불가능

이라고 단정하면 안 됩니다.

---

## 9. TIFF-to-PDF 사례와 연결

TIFF→PDF 최적화에서도 비슷한 사고방식을 사용했습니다.

기존:

```text
Memory → PNG temp file → File System → Memory → PDF
```

개선:

```text
Memory → MemoryStream → PDF
```

이것이 `sendfile`과 같은 기술은 아니지만 공통 원리는 같습니다.

> 필요하지 않은 중간 데이터 이동과 I/O 경로를 제거한다.

---

## 10. 60초 면접 답변

> Zero-copy는 데이터를 전송할 때 불필요한 메모리 복사, 특히 User Space와 Kernel Space 사이의 복사를 줄이는 최적화 기법을 말합니다. 일반적인 파일 전송에서는 파일 데이터를 User buffer로 읽은 뒤 다시 Socket을 통해 Kernel로 넘길 수 있는데, `sendfile` 같은 API를 사용하면 Kernel이 파일 데이터와 네트워크 전송 경로를 직접 연결할 수 있습니다. 이를 통해 CPU copy 비용과 memory bandwidth 사용을 줄일 수 있습니다. 다만 zero-copy가 물리적으로 어떤 데이터 이동도 없다는 뜻은 아니며, 실제 구현에서는 DMA나 Page Cache 같은 메커니즘이 함께 사용됩니다.

---

## 핵심 요약

```text
zero-copy = 불필요한 data copy 감소
sendfile  = file → socket 전송을 Kernel 내부에서 효율화
목표       = CPU copy + memory bandwidth + boundary overhead 감소
```

다음 주제: Socket Internals
