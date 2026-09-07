# 05. Load Balancing and Horizontal Scaling — Interview Drills

상태: **답변 대기**

## Questions

1. Vertical Scaling과 Horizontal Scaling의 차이를 설명하세요.
2. Load Balancer의 핵심 역할은 무엇인가요?
3. L4 Load Balancing과 L7 Load Balancing의 차이를 설명하세요.
4. Round Robin이 항상 실제 부하를 균등하게 만들지 못하는 이유는 무엇인가요?
5. Liveness와 Readiness의 차이는 무엇인가요?
6. Stateless Service가 Load Balancing과 잘 맞는 이유는 무엇인가요?
7. Auto Scaling에서 CPU 외에 어떤 지표를 볼 수 있나요?
8. App instance를 늘렸는데 오히려 DB 장애가 날 수 있는 이유는 무엇인가요?
9. `instance count × DB pool size`를 왜 확인해야 하나요?
10. Load Balancer 자체도 HA가 필요한 이유는 무엇인가요?
11. Connection Draining이 필요한 이유를 설명하세요.
12. Sticky Session의 장단점을 설명하세요.
13. CDN, Load Balancer, Cache, Queue는 각각 어떤 병목을 줄이나요?
14. App instance 하나가 죽었을 때 traffic은 어떻게 처리되어야 하나요?
15. 전체 App CPU는 낮지만 latency가 높을 때 어떤 병목을 의심할 수 있나요?

## Evaluation Criteria

- vertical/horizontal scaling을 정확히 구분하는가
- L4/L7와 routing algorithm을 설명하는가
- health check와 readiness를 운영 관점에서 설명하는가
- statelessness와 horizontal scaling을 연결하는가
- app scale-out이 downstream capacity를 압박할 수 있음을 이해하는가
- graceful shutdown/connection draining을 설명하는가
