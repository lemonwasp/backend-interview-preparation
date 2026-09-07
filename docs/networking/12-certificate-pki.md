# 12. Certificate와 PKI

## 학습 목표

- Certificate가 무엇을 증명하는지 설명할 수 있다.
- CA, Root CA, Intermediate CA의 역할을 설명할 수 있다.
- Certificate Chain 검증 과정을 설명할 수 있다.
- Domain Name과 Certificate의 관계를 설명할 수 있다.
- 만료, 폐기, 신뢰 저장소 문제를 실무 장애와 연결할 수 있다.

---

## 1. 인증서는 무엇인가?

인증서는 대략 다음 사실을 서명된 형태로 담은 문서입니다.

```text
이 공개키는 example.com이라는 이름과 연결되어 있으며
신뢰할 수 있는 발급자가 이를 확인했다.
```

인증서에는 보통 다음 정보가 들어갑니다.

- Subject / Domain 정보
- 공개키
- 유효 기간
- 발급자
- 서명 알고리즘
- CA의 디지털 서명

핵심은 **인증서 자체가 비밀키를 담는 것이 아니라 공개키와 신원을 연결한다**는 점입니다.

---

## 2. PKI란 무엇인가?

PKI(Public Key Infrastructure)는 공개키의 신뢰를 관리하는 전체 체계입니다.

주요 구성:

- Root CA
- Intermediate CA
- Server Certificate
- Browser / OS Trust Store
- 발급·갱신·폐기 절차

신뢰는 보통 다음 체인으로 이어집니다.

```text
Root CA
  ↓ signs
Intermediate CA
  ↓ signs
example.com Certificate
```

브라우저나 OS는 Root CA를 미리 신뢰하고 있기 때문에 그 Root가 서명한 Intermediate, 그리고 Intermediate가 서명한 Server Certificate까지 검증할 수 있습니다.

---

## 3. Certificate Chain 검증

클라이언트는 대략 다음을 확인합니다.

1. 인증서 서명이 올바른가?
2. Chain이 신뢰하는 Root CA까지 이어지는가?
3. 인증서가 아직 유효한 기간인가?
4. 현재 접속 Domain이 SAN(Subject Alternative Name)에 포함되는가?
5. 필요한 Key Usage가 허용되는가?
6. 폐기된 인증서인지 확인할 필요가 있는가?

이 중 하나라도 문제가 있으면 TLS 연결이 실패할 수 있습니다.

---

## 4. Root CA는 왜 특별한가?

Root CA는 자기 자신이 서명한 Self-signed Certificate를 가질 수 있습니다.

그렇다면 왜 믿을까요?

Root의 신뢰는 서명 체인 밖에서 시작됩니다. Browser나 OS Vendor가 Trust Store에 미리 포함해 배포하기 때문입니다.

즉:

> Certificate Chain은 무한히 올라가는 것이 아니라, 최종적으로 로컬 Trust Store의 신뢰 Anchor에서 끝난다.

---

## 5. Intermediate CA를 쓰는 이유

Root Private Key가 자주 사용되면 노출 위험이 커집니다.

그래서 Root CA는 보통 오프라인에 가깝게 보호하고, 실제 Server Certificate 발급은 Intermediate CA가 담당하도록 계층화합니다.

```text
Root Key: 매우 강하게 보호
Intermediate: 실무 발급 담당
Leaf Cert: 실제 서비스가 사용
```

Intermediate가 손상되어도 Root 전체를 폐기하지 않고 해당 Intermediate만 대응할 수 있다는 운영상 장점도 있습니다.

---

## 6. 인증서 장애

백엔드 실무에서 인증서는 흔한 장애 원인입니다.

- Certificate Expiration
- 잘못된 Domain / SAN
- Intermediate Certificate 누락
- Private Key와 Certificate 불일치
- Trust Store에 Root가 없음
- 시스템 시계 오류

예를 들어 서버 인증서가 정상이어도 Intermediate Chain을 제대로 제공하지 않으면 일부 Client는 검증에 실패할 수 있습니다.

---

## 7. 인증서 갱신 자동화

인증서 만료는 예측 가능한 장애입니다.

따라서 운영 환경에서는 다음이 중요합니다.

- 자동 발급 / 갱신
- 만료일 Monitoring
- Secret / Private Key 보호
- 배포 후 실제 TLS 검증

Let's Encrypt와 ACME 같은 자동화 체계가 널리 쓰이는 이유도 여기에 있습니다.

---

## 8. 60초 면접 답변

> Certificate는 공개키와 Domain 같은 신원 정보를 연결하고 CA가 디지털 서명한 문서입니다. 클라이언트는 서버 인증서에서 Intermediate CA, Root CA까지 Certificate Chain을 검증하고, 최종 Root가 로컬 Trust Store에 있는지 확인합니다. 또한 유효 기간과 접속 Domain이 SAN에 포함되는지도 확인합니다. Root CA는 직접 자주 인증서를 발급하기보다 보안을 위해 Intermediate CA를 사용하며, 실무에서는 인증서 만료나 Intermediate 누락 같은 문제도 TLS 장애의 흔한 원인이 됩니다.

---

## 핵심 요약

```text
Certificate = Identity + Public Key + CA Signature
PKI = 공개키 신뢰를 관리하는 체계
Chain = Leaf → Intermediate → Trusted Root
```

다음 주제: Forward Proxy와 Reverse Proxy
