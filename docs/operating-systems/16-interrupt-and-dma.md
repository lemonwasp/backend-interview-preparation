# 16. Interrupt와 DMA

## 이번 학습 목표

- Interrupt가 왜 필요한지 설명한다.
- Polling과 Interrupt의 차이를 이해한다.
- DMA가 CPU의 데이터 복사 부담을 줄이는 방식을 설명한다.
- Network/Disk I/O가 CPU와 장치 사이에서 어떻게 진행되는지 큰 흐름을 이해한다.

---

## 1. CPU가 장치를 계속 확인해야 한다면?

CPU가 디스크나 네트워크 카드에 작업을 시킨 뒤 매 순간

> 끝났어? 끝났어? 끝났어?

라고 계속 확인한다면 CPU 시간이 낭비됩니다.

이런 방식이 Polling입니다.

반대로 장치가 작업 완료나 이벤트 발생을 CPU에 알려주는 방식이 Interrupt입니다.

```text
CPU: 장치에게 작업 요청
CPU: 다른 일 수행
Device: 작업 완료
Device → Interrupt
CPU: 이벤트 처리
```

---

## 2. Interrupt란?

Interrupt는 CPU의 정상 실행 흐름에 이벤트가 발생했음을 알려
운영체제가 적절한 처리 루틴을 실행하게 만드는 메커니즘입니다.

대표적인 원인:

- Network packet 도착
- Disk I/O 완료
- Timer
- Keyboard 입력

Interrupt가 오면 CPU는 현재 실행 상태를 보존하고 Kernel의 Interrupt Handler 쪽으로 진입합니다.

---

## 3. Polling vs Interrupt

### Polling

```text
while (true) {
    장치 상태 확인
}
```

장점:
- 매우 빈번한 이벤트에서 예측 가능한 처리 가능
- 특정 고성능 환경에서는 유리할 수 있음

단점:
- 이벤트가 없어도 CPU를 소비할 수 있음

### Interrupt

장점:
- 이벤트가 없을 때 CPU를 다른 작업에 사용할 수 있음

단점:
- 이벤트가 너무 많으면 Interrupt 처리 자체가 부담이 될 수 있음

따라서 실제 고성능 시스템은 Interrupt와 Polling을 상황에 따라 조합하기도 합니다.

---

## 4. DMA가 필요한 이유

장치에서 메모리로 큰 데이터를 옮길 때 CPU가 바이트 하나하나를 직접 복사하면 비효율적입니다.

DMA(Direct Memory Access)는 장치와 메모리 사이의 데이터 전송을 전용 하드웨어가 수행하도록 합니다.

개념적으로:

```text
CPU
 │  전송 설정
 v
DMA / Device
 │
 ├──────────────→ RAM
 │   data transfer
 └─ 완료 후 interrupt
```

CPU는 전송을 설정한 뒤 다른 작업을 수행할 수 있습니다.

---

## 5. Network packet 수신 예

단순화하면:

```text
NIC receives packet
      ↓
DMA로 packet data를 RAM buffer에 기록
      ↓
Interrupt 또는 polling 기반 알림
      ↓
Kernel network stack 처리
      ↓
Socket receive buffer
      ↓
Application read/recv
```

실제 구현은 더 복잡하지만 중요한 포인트는
**NIC가 받은 데이터를 CPU가 직접 한 바이트씩 복사하지 않는다는 것**입니다.

---

## 6. Disk I/O에서도 비슷하다

SSD에 read 요청을 보냈다고 가정합니다.

```text
Application
  ↓ read
Kernel
  ↓ device request
Storage controller
  ↓ DMA
RAM
  ↓ completion interrupt
Kernel
  ↓ wake waiting task
Application
```

이 흐름은 Blocking/Async I/O, Page Cache와도 연결됩니다.

---

## 7. Interrupt Storm

이벤트가 지나치게 많아 Interrupt 처리에 CPU 시간이 과도하게 사용되는 상황을 Interrupt Storm이라고 부를 수 있습니다.

네트워크 트래픽이 매우 많은 서버에서는 packet 하나마다 단순 Interrupt를 발생시키는 것보다
여러 packet을 묶어 처리하거나 polling을 활용하는 전략이 더 효율적일 수 있습니다.

Linux 네트워크의 NAPI 같은 구조가 이런 문제와 연결됩니다.

---

## 8. 왜 백엔드 개발자가 알아야 할까?

백엔드 개발자는 보통 DMA를 직접 프로그래밍하지 않습니다.
하지만 다음을 이해하는 데 중요합니다.

- I/O는 CPU 계산과 다른 성격의 작업이다.
- async I/O가 가능한 이유
- packet이 NIC에서 애플리케이션까지 오는 과정
- zero-copy가 왜 의미가 있는지
- CPU utilization이 낮아도 I/O bottleneck이 생길 수 있는 이유

---

## 9. 60초 면접 답변

> Interrupt는 장치나 Timer 같은 이벤트가 발생했음을 CPU에 알려 운영체제가 적절한 처리 루틴을 실행하게 하는 메커니즘입니다. CPU가 장치 상태를 계속 확인하는 Polling과 달리, 이벤트가 없을 때 CPU를 다른 작업에 사용할 수 있습니다. DMA는 디스크나 NIC와 메모리 사이의 데이터 전송을 CPU가 직접 복사하지 않고 전용 하드웨어가 처리하게 해 CPU 부담을 줄입니다. 예를 들어 NIC가 packet을 받으면 DMA로 RAM에 데이터를 옮기고, Interrupt나 polling을 통해 Kernel이 이를 처리한 뒤 Socket buffer를 거쳐 애플리케이션에 전달합니다.

---

## 핵심 요약

```text
Polling   = CPU가 장치 상태를 직접 반복 확인
Interrupt = 장치/이벤트가 CPU에 알림
DMA       = 장치 ↔ RAM 데이터 전송을 CPU 대신 수행
```

다음 주제: Zero-copy와 sendfile
