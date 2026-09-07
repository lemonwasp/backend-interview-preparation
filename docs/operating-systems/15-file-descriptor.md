# 15. File Descriptor

## 이번 학습 목표

- File Descriptor가 무엇인지 설명한다.
- 파일, Socket, Pipe가 왜 비슷한 인터페이스로 다뤄지는지 이해한다.
- Process별 Descriptor Table과 Kernel Object의 관계를 설명한다.
- `open → read/write → close` 흐름을 이해한다.
- FD Leak이 백엔드 장애로 이어지는 이유를 설명한다.

---

## 1. File Descriptor를 번호표라고 생각해보자

Linux/Unix 계열에서 Process가 파일이나 Socket 같은 Kernel 자원을 사용할 때,
애플리케이션은 Kernel 내부 구조체를 직접 만지지 않습니다.

대신 작은 정수 번호를 받습니다.

```text
3 → log file
4 → database socket
5 → client socket
```

이 번호가 File Descriptor(FD)입니다.

> File Descriptor는 Process가 열린 Kernel 자원을 가리키기 위해 사용하는 작은 정수 핸들이다.

---

## 2. Process 안에는 Descriptor Table이 있다

개념적으로:

```text
Process
  FD table
  0 → stdin
  1 → stdout
  2 → stderr
  3 → open file description
  4 → socket object
```

FD 자체는 파일 데이터가 아닙니다.
FD는 Process의 테이블 안에서 Kernel이 관리하는 열린 자원을 가리키는 인덱스에 가깝습니다.

같은 숫자 `3`이라도 Process A의 FD 3과 Process B의 FD 3은 전혀 다른 자원을 가리킬 수 있습니다.

---

## 3. 왜 파일뿐 아니라 Socket도 FD인가?

Unix 철학에서는 많은 I/O 자원을 비슷한 방식으로 추상화합니다.

- regular file
- socket
- pipe
- terminal
- device

이런 자원들은 `read`, `write`, `close` 같은 공통 인터페이스로 다룰 수 있습니다.

이 덕분에 애플리케이션은 각 장치의 저수준 동작을 매번 직접 알 필요가 없습니다.

---

## 4. open부터 close까지

```text
open("app.log")
      ↓
Kernel이 파일을 연다
      ↓
Process FD table에 등록
      ↓
FD 3 반환
      ↓
write(3, ...)
      ↓
close(3)
```

`close`가 중요한 이유는 FD가 무한하지 않기 때문입니다.

---

## 5. FD Leak이란?

파일이나 Socket을 계속 열고 닫지 않으면 Process가 보유한 FD가 쌓입니다.

```text
요청 1 → socket open → close 안 함
요청 2 → socket open → close 안 함
...
```

결국 Process 또는 시스템의 FD limit에 도달하면 새 파일이나 새 네트워크 연결을 열 수 없게 됩니다.

백엔드에서는 다음과 같은 장애로 나타날 수 있습니다.

- 새 DB 연결 실패
- 새 HTTP connection accept 실패
- 로그 파일 열기 실패
- `Too many open files`

---

## 6. Socket Pool과 FD

DB Connection Pool의 연결 하나는 보통 내부적으로 Socket을 포함하고,
Unix 계열에서는 그 Socket도 FD로 표현됩니다.

따라서 Connection Pool 크기를 무작정 키우는 것은 단순 메모리 문제가 아닙니다.

- FD 사용량
- DB 서버 connection limit
- Thread/Task 수
- Buffer 메모리

등과 같이 봐야 합니다.

---

## 7. dup와 공유

`dup` 계열 호출은 기존 FD와 같은 열린 파일 상태를 가리키는 새 FD를 만들 수 있습니다.

이때 단순히 파일 이름만 같은 것이 아니라 Kernel 내부의 열린 상태를 공유할 수 있어
파일 offset 등의 동작을 이해할 때 주의해야 합니다.

---

## 8. fork와 FD

Unix에서 `fork()` 후 자식 Process는 부모의 열린 FD를 상속할 수 있습니다.

이 때문에 서버 프로그램에서 의도치 않은 FD 상속은 자원 정리나 종료 조건을 복잡하게 만들 수 있습니다.

실무에서는 close-on-exec 같은 설정도 중요합니다.

---

## 9. C#에서는 왜 FD가 잘 안 보일까?

C#에서는 보통 다음처럼 고수준 객체를 사용합니다.

```csharp
using var stream = File.OpenRead(path);
```

개발자는 `FileStream`, `Socket`, `NetworkStream` 같은 객체를 다루지만
Linux 위에서 실행될 경우 내부적으로 OS FD를 사용할 수 있습니다.

따라서 `using` / `Dispose()`를 무시하면 결국 OS 자원 leak으로 연결될 수 있습니다.

---

## 10. 60초 면접 답변

> File Descriptor는 Unix 계열 운영체제에서 Process가 파일, Socket, Pipe 같은 열린 Kernel 자원을 참조하기 위해 사용하는 작은 정수 핸들입니다. Process마다 Descriptor Table이 있고, FD는 그 테이블을 통해 Kernel의 실제 열린 자원을 가리킵니다. 파일뿐 아니라 Socket도 같은 read/write/close 인터페이스로 다룰 수 있어 I/O 추상화가 단순해집니다. 백엔드에서는 FD를 닫지 않으면 `Too many open files` 같은 장애가 발생할 수 있기 때문에 파일 스트림, Socket, DB connection 같은 자원을 반드시 적절히 해제해야 합니다.

---

## 핵심 요약

```text
FD = Process-local integer handle
FD → Kernel-managed open resource
file/socket/pipe can share similar I/O interface
close 누락 → FD leak → production failure
```

다음 주제: Interrupt와 DMA
