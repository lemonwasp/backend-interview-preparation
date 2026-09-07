# 30. Networking Review — 60-second Answer Checkpoint

## 목적

이 문서는 새 이론을 추가하지 않는다.

Networking 01~29를 기술면접에서 실제로 말할 수 있도록 **핵심 축으로 압축**한다.

완료 기준은 문서를 읽는 것이 아니라 다음 질문을 자료 없이 60초 안에 설명하는 것이다.

---

# A. TCP / Transport

## 1. TCP와 UDP 차이

> TCP는 reliable ordered byte stream이고 UDP는 connectionless datagram 기반 전송입니다. TCP는 순서 보장, 재전송, 흐름 제어, 혼잡 제어를 제공하는 대신 연결 관리와 상태 유지 비용이 있습니다. UDP는 이러한 기능을 기본 제공하지 않아 overhead가 작지만 reliability가 필요하면 application 또는 상위 protocol이 직접 구현해야 합니다. 그래서 일반적인 HTTP/1.1·2는 TCP를 사용하고 QUIC은 UDP 위에 reliability와 congestion control을 다시 구현합니다.

핵심 꼬리질문:
- UDP가 항상 TCP보다 빠른가?
- QUIC이 UDP를 쓰는데 어떻게 reliable한가?

## 2. 3-way Handshake

> TCP는 SYN, SYN/ACK, ACK의 3-way handshake로 양방향 통신 준비와 initial sequence number를 동기화합니다. 새 연결에는 RTT 비용이 발생하므로 HTTP keep-alive와 connection pooling으로 연결을 재사용하는 것이 중요합니다. SYN 요청만 대량으로 보내 자원을 소모시키는 SYN flood 같은 공격도 이 연결 상태 관리와 관련 있습니다.

## 3. TIME_WAIT

> TIME_WAIT은 보통 active closer가 마지막 ACK를 재전송할 수 있게 하고 이전 connection의 지연 segment가 새 connection에 섞이는 위험을 줄이기 위한 상태입니다. 많은 TIME_WAIT이 보인다고 무조건 제거할 것이 아니라 connection churn이 왜 큰지 먼저 봐야 하며 keep-alive와 pooling으로 불필요한 연결 생성·종료를 줄이는 것이 기본입니다.

## 4. Retransmission / RTO

> TCP는 Sequence Number와 ACK를 이용해 손실을 감지하고 재전송합니다. RTO는 RTT를 기반으로 계산되는 timeout이고, duplicate ACK가 반복되면 timeout을 기다리지 않고 fast retransmit을 할 수 있습니다. Packet loss가 있으면 평균 latency보다 p95/p99 latency가 크게 악화될 수 있으며 application retry와 TCP retransmission은 서로 다른 계층의 복구입니다.

## 5. Flow Control vs Congestion Control

> Flow Control은 receiver가 감당할 수 있는 만큼만 보내도록 `rwnd`로 receiver를 보호하고, Congestion Control은 network 자체가 과부하되지 않도록 `cwnd`를 조절합니다. 실제 sender의 전송 가능량은 두 제한 중 더 작은 쪽의 영향을 받습니다.

---

# B. DNS / HTTP / QUIC / TLS

## 6. DNS

> DNS는 domain name을 IP 등 resource record로 변환합니다. Client는 보통 recursive resolver에 질의하고 resolver가 cache에 없으면 root, TLD, authoritative server를 따라가며 답을 찾습니다. TTL은 cache 유효 시간을 결정해 latency와 DNS traffic을 줄이지만 변경 전파를 늦출 수 있습니다. DNS는 보통 UDP를 사용하지만 큰 응답이나 특정 상황에서는 TCP도 사용할 수 있습니다.

## 7. HTTP/1.1 vs HTTP/2

> HTTP/1.1은 persistent connection으로 connection churn을 줄일 수 있지만 한 connection에서 여러 요청을 효율적으로 병렬 처리하는 데 한계가 있습니다. HTTP/2는 binary framing과 stream multiplexing으로 application-level HOL 문제를 크게 줄였지만 underlying TCP에서 packet loss가 발생하면 TCP-level HOL은 남습니다.

## 8. HTTP/3 / QUIC

> HTTP/3는 QUIC 위에서 동작하고 QUIC은 UDP 위에 reliability, congestion control, encryption을 통합합니다. Stream별 전송 상태를 분리해 하나의 stream loss가 다른 stream 전체를 TCP처럼 막는 영향을 줄일 수 있습니다. Connection ID를 이용해 network path가 바뀌어도 connection migration을 지원할 수 있고 TLS 1.3이 protocol에 통합되어 있습니다. 다만 모든 환경에서 항상 더 빠른 것은 아닙니다.

## 9. TLS / HTTPS

> HTTPS는 HTTP를 TLS로 보호하는 구조입니다. TLS는 confidentiality, integrity, authentication을 제공하고 handshake에서 server certificate를 검증하며 key agreement를 수행한 뒤 application data는 효율적인 symmetric encryption으로 보호합니다. TLS termination이 load balancer나 reverse proxy에서 일어날 수도 있으므로 backend까지의 암호화 경계도 설계해야 합니다.

## 10. Certificate / PKI

> Certificate 검증은 보통 leaf certificate에서 intermediate를 거쳐 trusted root까지 chain을 확인하고 hostname, validity, signature 등을 검증합니다. Root CA에 대한 신뢰는 client의 trust store에서 시작합니다. 인증서 만료나 intermediate 누락은 실제 서비스 장애 원인이 될 수 있습니다.

---

# C. Proxy / Load Balancing / Cache

## 11. Forward Proxy vs Reverse Proxy

> Forward Proxy는 client를 대신해 외부 서버에 요청하고, Reverse Proxy는 server 앞에서 client 요청을 받아 backend로 전달합니다. Reverse Proxy는 TLS termination, routing, caching, compression, authentication 같은 공통 기능을 맡을 수 있습니다. `X-Forwarded-For` 같은 header는 trusted proxy boundary를 고려하지 않으면 spoofing 위험이 있습니다.

## 12. L4 vs L7 Load Balancing

> L4 Load Balancer는 주로 IP와 port 같은 transport 정보를 기반으로 분산하고, L7 Load Balancer는 HTTP host, path, header 같은 application 정보를 이용해 더 세밀하게 routing할 수 있습니다. Round Robin, Least Connections, Weighted, Hash 등의 알고리즘이 있고 health check와 load balancer 자체의 HA도 필요합니다.

## 13. HTTP Cache / CDN

> HTTP Cache는 origin까지 가지 않고 저장된 response를 재사용해 latency와 origin load를 줄입니다. Freshness와 Validation은 다르고, ETag를 사용하면 stale resource를 조건부 요청으로 검증해 304를 받을 수 있습니다. `no-cache`는 저장 금지가 아니라 재사용 전 검증 요구이고 `no-store`는 저장 자체를 막는 의미입니다. Shared cache에서는 cache key와 인증 경계를 잘못 설계하면 사용자 데이터 노출이 생길 수 있습니다.

---

# D. Realtime / RPC / Gateway

## 14. WebSocket vs SSE

> WebSocket은 장기 connection 위에서 full-duplex 양방향 통신을 제공해 chat이나 realtime collaboration에 적합합니다. SSE는 HTTP 기반 server-to-client 단방향 stream이라 구현이 더 단순하고 AI token streaming이나 notification에 적합합니다. WebSocket은 scale-out 시 어떤 node가 connection을 가지고 있는지와 cross-node message delivery를 고려해야 하고, SSE 자동 재연결이 exactly-once를 의미하지는 않습니다.

## 15. gRPC vs REST

> gRPC는 RPC framework이고 보통 Protocol Buffers와 HTTP/2를 이용해 schema 기반 code generation과 streaming, deadline, cancellation을 지원합니다. REST는 resource-oriented HTTP API 설계 스타일입니다. 핵심 차이는 JSON vs Binary가 아니라 resource-oriented와 method/action-oriented abstraction입니다. Public API에는 REST, 내부 service-to-service에는 gRPC를 함께 쓰는 구조가 자연스러울 수 있습니다.

## 16. API Gateway

> API Gateway는 외부 요청의 공통 진입점으로 routing, authentication, rate limiting, TLS termination, observability 같은 cross-cutting concern을 처리합니다. 하지만 domain authorization과 business logic을 모두 Gateway에 넣으면 coupling과 bottleneck이 커집니다. Gateway 자체도 SPOF가 될 수 있어 horizontal scaling과 health check가 필요하고 여러 계층이 각각 retry하면 retry amplification이 발생할 수 있습니다.

---

# E. Resilience

## 17. Rate Limiting

> Rate Limiting은 시스템 보호와 공정한 사용을 위해 요청 유입량을 제한합니다. Token Bucket은 일정 속도로 token을 채우면서 bucket 크기만큼 burst를 허용할 수 있고, distributed rate limit에서는 여러 instance가 동일한 counter/state를 어떻게 공유할지 고려해야 합니다.

## 18. Timeout Budget

> Timeout은 요청을 무한정 기다리지 않게 하지만 각 downstream timeout을 독립적으로 크게 잡으면 전체 latency budget을 초과합니다. 그래서 client request deadline에서 downstream별 budget을 나누고 남은 deadline을 전파해야 합니다. Timeout이 발생하면 가능하면 cancellation도 downstream에 전달해 이미 필요 없는 작업이 계속 자원을 쓰지 않게 해야 합니다.

## 19. Retry / Backoff / Jitter

> Retry는 transient failure 복구에 유용하지만 실패 중인 시스템에 추가 traffic을 보내므로 제한적으로 사용해야 합니다. Exponential Backoff는 retry 간격을 늘리고 Jitter는 여러 client가 동시에 재시도하는 herd를 분산합니다. 여러 계층이 독립적으로 retry하면 request 수가 기하급수적으로 늘 수 있으므로 retry layer와 budget을 명확히 정해야 합니다.

## 20. Circuit Breaker

> Circuit Breaker는 반복 실패가 일정 기준을 넘으면 Open 상태로 전환해 downstream 호출을 fail fast하고, 일정 시간이 지나면 Half-Open에서 소수 요청으로 회복 여부를 확인합니다. Timeout은 개별 호출 대기 시간을 제한하고 Circuit Breaker는 반복 장애 시 호출 자체를 잠시 차단한다는 차이가 있습니다.

## 21. Bulkhead / Backpressure

> Bulkhead는 thread pool, connection pool, semaphore, queue 같은 자원을 분리해 한 dependency의 장애가 전체 시스템 자원을 고갈시키지 못하게 합니다. Backpressure는 producer가 consumer capacity를 초과할 때 bounded queue, concurrency limit, load shedding 등을 통해 upstream 속도를 조절합니다. 무한 queue는 장애를 해결하지 않고 latency와 memory 사용만 키울 수 있습니다.

## 22. Idempotency Key

> Idempotency Key는 같은 business request가 network timeout이나 retry로 여러 번 도착해도 한 번만 처리한 것처럼 만들기 위한 key입니다. Server는 최초 처리 결과를 key와 저장하고 같은 key가 오면 기존 결과를 재사용합니다. 결제와 주문 같은 non-idempotent operation의 retry에서 중요하며 unique constraint와 request fingerprint로 동시 요청과 key misuse를 막아야 합니다.

---

# F. Connection Resource Management

## 23. Connection Pool

> Connection Pool은 DB나 HTTP connection을 재사용해 handshake와 인증 비용을 줄이고 동시에 사용하는 connection 수를 제한합니다. 너무 작으면 acquisition wait가 커지고 너무 크면 downstream capacity를 넘겨 contention을 키울 수 있습니다. 전체 application instance 수를 포함해 pool size를 계산하고 active, idle, wait time, timeout, leak을 모니터링해야 합니다.

## 24. NAT / Ephemeral Port

> Client outbound TCP connection은 ephemeral source port를 사용하고 NAT 환경에서는 private tuple이 public IP와 port mapping으로 변환됩니다. Connection churn이 크면 TIME_WAIT, ephemeral port, NAT mapping pressure가 증가해 새 connection 생성 실패로 이어질 수 있습니다. 그래서 keep-alive와 connection pooling이 중요하고 장애 시 process 내부뿐 아니라 NAT gateway와 connect error 지표도 봐야 합니다.

---

# 10개 필수 비교 질문

다음 10개는 즉답할 수 있어야 한다.

1. TCP vs UDP
2. Flow Control vs Congestion Control
3. HTTP/1.1 vs HTTP/2 vs HTTP/3
4. Forward Proxy vs Reverse Proxy
5. L4 vs L7 Load Balancer
6. WebSocket vs SSE
7. REST vs gRPC/RPC
8. Timeout vs Circuit Breaker
9. Rate Limiting vs Backpressure
10. Thread Pool vs Connection Pool

# 5개 장애 시나리오

## 시나리오 1 — 외부 API latency 폭증

확인 순서:

```text
DNS
-> TCP connect
-> TLS handshake
-> Connection pool wait
-> Remote processing
-> Timeout
-> Retry rate
-> Circuit breaker
```

## 시나리오 2 — DB Pool Exhaustion

```text
slow query / long transaction / leak
-> connection occupancy 증가
-> pool wait 증가
-> request timeout
-> retry
-> DB 부하 증가
```

## 시나리오 3 — 503 폭증

```text
Load balancer health
-> App saturation
-> Queue / concurrency
-> downstream failure
-> rate limit / load shedding
```

## 시나리오 4 — Connect Failure 증가

```text
DNS
-> route / firewall
-> ephemeral ports
-> NAT mapping
-> remote connection limit
-> retry amplification
```

## 시나리오 5 — p99만 급격히 악화

확인:

- Packet loss / retransmission
- Queue wait
- pool acquisition
- downstream tail latency
- GC / CPU saturation
- Retry

# 최종 완료 조건

Networking 트랙은 다음을 만족할 때 완료로 올린다.

- 위 24개 핵심 답변을 대부분 60초 이내 설명
- 10개 비교 질문에 자료 없이 답변
- 5개 장애 시나리오에서 조사 순서를 설명
- Quiz에서 틀린 내용을 문서에 반영
- 최소 D+1 재시험 수행

문서가 30개 존재하는 것만으로 완료 처리하지 않는다.
