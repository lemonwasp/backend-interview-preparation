# 02. 사용자 모드, 커널 모드, System Call

## 이번 학습 목표

이 문서를 학습한 뒤 다음을 자신의 말로 설명할 수 있어야 합니다.

- 왜 CPU 권한을 User Mode와 Kernel Mode로 나누는가?
- 애플리케이션은 커널 기능을 어떻게 요청하는가?
- System Call이 호출될 때 내부에서 어떤 일이 일어나는가?
- Mode Switch와 Context Switch는 무엇이 다른가?
- System Call 비용이 백엔드 성능과 어떤 관계가 있는가?

---

## 1. 컴퓨터를 보안 구역이 있는 회사라고 생각해보자

회사를 하나 떠올려봅시다.

일반 직원은 자신의 자리에서 문서를 작성하고 프로그램을 사용할 수 있습니다.
하지만 서버실 전원을 끄거나, 다른 직원의 서랍을 열거나, 회사 전체 네트워크
장비를 직접 바꾸는 권한은 없습니다.

반대로 시스템 관리자는 제한된 절차를 거쳐 더 강한 권한을 사용할 수 있습니다.

컴퓨터도 비슷합니다.

```text
일반 애플리케이션
      ↓
   User Mode
      ↓  System Call
   Kernel Mode
      ↓
CPU / Memory / Disk / Network Device
```

애플리케이션은 보통 **User Mode**에서 실행되고, 운영체제의 핵심인 Kernel은
**Kernel Mode**에서 실행됩니다.

한 문장으로 정리하면:

> User Mode는 애플리케이션의 권한을 제한해 시스템을 보호하고,
> Kernel Mode는 운영체제가 CPU·메모리·장치를 관리할 수 있도록 높은 권한을
> 제공한다.

---

## 2. 왜 권한을 굳이 나눌까?

만약 모든 프로그램이 Kernel과 같은 권한으로 실행된다면 단순한 버그 하나도
시스템 전체 장애로 이어질 수 있습니다.

예를 들어 잘못 작성된 프로그램이 다음 작업을 할 수 있다고 생각해봅시다.

- 다른 Process의 메모리를 수정한다.
- 운영체제 코드가 있는 메모리를 덮어쓴다.
- 디스크 장치를 직접 잘못 제어한다.
- 인터럽트 설정이나 CPU 제어 정보를 마음대로 바꾼다.

이 경우 하나의 애플리케이션 오류가 다른 프로그램과 운영체제까지 망가뜨릴 수
있습니다.

따라서 현대 운영체제는 애플리케이션과 커널의 권한을 분리합니다.

### User Mode

일반 애플리케이션이 실행되는 제한된 권한 영역입니다.

- 임의의 커널 메모리에 접근할 수 없다.
- 특정 privileged instruction을 직접 실행할 수 없다.
- 장치를 마음대로 직접 제어할 수 없다.
- 운영체제가 허용한 인터페이스를 통해 기능을 요청해야 한다.

### Kernel Mode

운영체제 Kernel이 실행되는 높은 권한 영역입니다.

- CPU Scheduling
- Virtual Memory 관리
- Process와 Thread 관리
- File System 처리
- Device Driver 실행
- Network Stack 처리

등 시스템 핵심 작업을 수행합니다.

---

## 3. 그렇다면 애플리케이션은 파일을 어떻게 읽을까?

User Mode 프로그램은 디스크를 직접 제어할 수 없습니다.
그렇다고 파일을 못 읽는 것은 아닙니다.

애플리케이션은 운영체제에 요청합니다.

```text
애플리케이션
"이 파일의 데이터를 읽어 주세요."
        ↓
System Call
        ↓
Kernel
- 권한 확인
- File System 확인
- Cache 확인
- 필요하면 Device I/O 요청
        ↓
결과 반환
        ↓
애플리케이션
```

이처럼 **User Mode 프로그램이 Kernel의 기능을 요청하는 공식 인터페이스**가
System Call입니다.

대표적인 System Call의 역할은 다음과 같습니다.

| 목적 | 예시 |
|---|---|
| 파일 | open, read, write, close |
| Process | fork, exec, exit, wait |
| 메모리 | mmap 등 |
| 네트워크 | socket, bind, connect, send, recv |
| 동기화·대기 | futex, poll, epoll 등 |

구체적인 이름과 동작은 운영체제마다 다릅니다.

Windows, Linux, macOS가 모두 같은 System Call 목록을 사용하는 것은 아닙니다.

---

## 4. 라이브러리 함수와 System Call은 같은 것일까?

항상 같지는 않습니다.

개발자는 보통 System Call을 직접 호출하기보다 언어 Runtime이나 표준
라이브러리의 API를 사용합니다.

예를 들어 C#에서 파일을 읽는다고 생각해봅시다.

```csharp
var bytes = File.ReadAllBytes(path);
```

개발자 입장에서는 `File.ReadAllBytes`라는 .NET API를 호출합니다.
하지만 실제 파일 데이터가 필요하다면 결국 Runtime과 운영체제 API를 거쳐
Kernel의 파일 처리 기능으로 내려갑니다.

```text
C# code
  ↓
.NET API / Runtime
  ↓
OS API
  ↓
System Call boundary
  ↓
Kernel
```

중요한 점은 다음입니다.

> 라이브러리 함수 호출 = System Call 1회

라고 단순하게 생각하면 안 됩니다.

라이브러리 내부에서 버퍼링을 하거나 여러 System Call을 묶을 수도 있고,
반대로 하나의 고수준 API가 여러 운영체제 요청을 만들 수도 있습니다.

---

## 5. System Call이 발생하면 무슨 일이 일어날까?

개념적으로는 다음 순서로 이해하면 됩니다.

### 1) User Mode에서 프로그램 실행

애플리케이션 코드가 일반 권한으로 실행됩니다.

### 2) Kernel 기능이 필요해짐

예를 들어 파일 읽기, 네트워크 송신, 메모리 매핑 같은 작업이 필요합니다.

### 3) System Call 진입

CPU가 정해진 System Call 진입 명령과 규칙을 통해 Kernel Mode로 전환합니다.

### 4) Kernel이 요청 검증

운영체제는 요청 번호, 인자, 메모리 주소, 권한 등을 확인합니다.

잘못된 User Pointer나 허용되지 않은 요청을 그대로 신뢰해서는 안 됩니다.

### 5) Kernel 작업 수행

File System, Scheduler, Network Stack, Driver 등 필요한 Kernel 기능이 실행됩니다.

### 6) 결과 반환

작업 결과나 오류 코드를 User Mode 프로그램에 반환하고 애플리케이션 실행을
이어갑니다.

간단히 그리면:

```text
User Mode
   |
   | system call
   v
Kernel Mode
   |
   | validate + work
   v
Kernel Mode
   |
   | return
   v
User Mode
```

---

## 6. Mode Switch와 Context Switch는 다르다

면접에서 자주 혼동하는 부분입니다.

### Mode Switch

CPU 실행 권한이 User Mode와 Kernel Mode 사이에서 바뀌는 것입니다.

예:

```text
같은 Thread
User Mode → Kernel Mode → User Mode
```

System Call을 처리한 뒤 같은 Thread로 바로 돌아오면 Mode Switch는 있었지만
다른 Thread로 교체되지 않았을 수 있습니다.

### Context Switch

CPU가 실행하던 Process 또는 Thread를 멈추고 다른 실행 흐름으로 전환하는
것입니다.

이때 현재 실행 상태를 저장하고 다른 작업의 상태를 복원해야 합니다.

```text
Thread A
   ↓ 상태 저장
Scheduler
   ↓ 상태 복원
Thread B
```

따라서:

> System Call은 보통 User/Kernel Mode 전환을 수반하지만,
> System Call 하나가 반드시 Context Switch 하나를 의미하지는 않는다.

예를 들어 `read()`가 Page Cache에서 즉시 데이터를 얻고 같은 Thread로
돌아온다면 다른 Thread로 전환할 필요가 없을 수 있습니다.

반대로 I/O를 기다려야 한다면 현재 Thread가 Block되고 Scheduler가 다른
Thread를 실행하면서 Context Switch가 발생할 수 있습니다.

---

## 7. System Call은 왜 비용이 있을까?

일반 함수 호출보다 Kernel 경계를 넘는 작업에는 추가 비용이 있습니다.

대표적으로:

- User Mode ↔ Kernel Mode 전환
- 요청 인자와 권한 검증
- Kernel 자료구조 접근
- User/Kernel 공간 사이 데이터 복사 가능성
- I/O 대기 가능성
- 경우에 따라 Scheduling과 Context Switch

하지만 다음처럼 과장하면 안 됩니다.

> “System Call은 무조건 느리다.”

System Call 자체의 경계 통과 비용도 존재하지만 실제 파일·네트워크 I/O에서는
장치 대기, 데이터 복사, 네트워크 지연 같은 비용이 훨씬 더 클 수도 있습니다.

따라서 성능을 볼 때는 **System Call 횟수만 세는 것이 아니라 전체 데이터 흐름과
I/O 패턴을 함께 봐야 합니다.**

---

## 8. 백엔드 서버에서는 어디에서 보일까?

백엔드 서버는 System Call과 매우 가까운 프로그램입니다.

HTTP 요청 하나를 처리한다고 생각해봅시다.

```text
Client
  ↓
Network Card
  ↓
Kernel Network Stack
  ↓
socket receive
  ↓
Backend Runtime
  ↓
Application Code
```

응답도 반대 방향으로 내려갑니다.

```text
Application Code
  ↓
socket send
  ↓
Kernel Network Stack
  ↓
Network Card
  ↓
Client
```

파일 로그를 남길 때도 File System 관련 운영체제 기능이 사용되고,
DB 연결도 결국 Socket I/O를 사용하며, Thread를 잠재우거나 깨우는 과정에도
운영체제 기능이 개입할 수 있습니다.

즉 백엔드 개발에서 다음 주제를 제대로 이해하려면 System Call 경계를 이해하는
것이 도움이 됩니다.

- Blocking / Non-blocking I/O
- Async I/O
- Thread Pool
- Socket
- File I/O
- Context Switching
- Performance Profiling

---

## 9. TIFF-to-PDF 사례와 다시 연결

기존 구현에서는 페이지마다 임시 PNG 파일을 생성하고 다시 읽고 삭제했습니다.

```text
Image conversion
  ↓
create/write temporary file
  ↓
read temporary file
  ↓
add to PDF
  ↓
delete temporary file
```

각 파일 API가 정확히 몇 번의 System Call로 연결되는지는 Runtime과 OS,
버퍼링 방식에 따라 달라집니다.

하지만 분명한 것은 파일 경로를 사용하는 동안 다음과 같은 운영체제 계층을
반복적으로 거칠 가능성이 커진다는 점입니다.

- File System API
- 파일 메타데이터 처리
- Buffer / Page Cache 처리
- User/Kernel 경계 통과
- 데이터 복사

`MemoryStream` 기반 방식으로 임시 파일 경로 자체를 제거하면 이런 파일 시스템
관련 작업을 줄일 수 있습니다.

이 사례의 핵심은 단순히

> “MemoryStream이 무조건 빠르다”

가 아니라,

> 불필요한 I/O 경로와 운영체제 상호작용을 제거해 전체 데이터 흐름을 짧게 했다.

라고 설명하는 것입니다.

---

## 10. 자주 하는 오해

### “Kernel Mode는 Kernel이라는 별도 CPU에서 실행된다”

아닙니다. 같은 CPU가 다른 권한 수준으로 코드를 실행하는 것입니다.

### “User Mode 프로그램은 운영체제 기능을 사용할 수 없다”

직접 높은 권한 명령을 수행할 수 없다는 뜻입니다. System Call과 운영체제 API를
통해 필요한 기능을 요청할 수 있습니다.

### “System Call이 발생하면 무조건 다른 Process로 바뀐다”

아닙니다. Mode Switch와 Context Switch는 다른 개념입니다.

### “File API 하나는 System Call 하나다”

반드시 그렇지 않습니다. Runtime, 버퍼링, 운영체제 구현에 따라 달라질 수
있습니다.

### “System Call을 줄이면 무조건 프로그램이 빨라진다”

System Call 수는 성능의 한 요소일 뿐입니다. 실제 병목은 Disk, Network,
Lock, GC, 알고리즘, 데이터 복사 등 다른 곳에 있을 수도 있습니다.

---

## 11. 60초 면접 답변

> 애플리케이션은 보통 User Mode에서 제한된 권한으로 실행되고, Kernel은
> Kernel Mode에서 CPU, 메모리, 파일 시스템과 장치 같은 핵심 자원을 관리합니다.
> 이렇게 권한을 나누는 이유는 애플리케이션의 버그나 악성 코드가 시스템 전체를
> 직접 손상시키는 것을 막기 위해서입니다. 애플리케이션이 파일이나 네트워크처럼
> Kernel 기능이 필요하면 System Call을 통해 요청합니다. 이 과정에서 CPU가
> Kernel Mode로 진입해 요청과 권한을 검증하고 작업한 뒤 다시 User Mode로
> 돌아옵니다. 이 Mode Switch는 다른 Thread로 바뀌는 Context Switch와는 다른
> 개념이며, System Call이 항상 Context Switch를 발생시키는 것은 아닙니다.

---

## 핵심 요약

```text
User Mode   = 제한된 권한으로 애플리케이션 실행
Kernel Mode = 높은 권한으로 OS 핵심 기능 실행
System Call = User Mode가 Kernel 기능을 요청하는 공식 통로
Mode Switch ≠ Context Switch
```

다음 주제:

- Process와 Thread
- Process Address Space
- Context Switching
