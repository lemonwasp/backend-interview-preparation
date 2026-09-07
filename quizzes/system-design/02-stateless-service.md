# 02. Stateless Service — Interview Drills

상태: **답변 대기**

## Questions

1. Stateful Service와 Stateless Service의 차이를 설명하세요.
2. Stateless Service가 horizontal scaling에 유리한 이유는 무엇인가요?
3. 세션을 특정 App Server의 memory에 저장하면 어떤 문제가 생기나요?
4. JWT를 사용하면 어떤 장단점이 있나요?
5. Shared Session Store를 Redis에 두면 어떤 비용이 추가되나요?
6. Sticky Session의 장점과 단점을 설명하세요.
7. Stateless가 시스템 전체에 state가 없다는 뜻이 아닌 이유는 무엇인가요?
8. Local Cache를 사용해도 stateless라고 할 수 있나요? 어떤 조건에서 주의해야 하나요?
9. 사용자가 업로드한 파일을 App Server local disk에만 저장하면 scale-out 시 어떤 문제가 생기나요?
10. Stateless architecture에서도 병목이 생길 수 있는 이유는 무엇인가요?
11. Stateful 구조에서 instance failure가 더 큰 영향을 줄 수 있는 이유는 무엇인가요?
12. 실시간 WebSocket 서비스는 stateless 설계와 어떤 긴장을 가지나요?

## Evaluation Criteria

- 특정 instance에 붙는 state와 shared/durable state를 구분하는가
- horizontal scaling과 load balancing을 연결하는가
- JWT/Redis/sticky session의 trade-off를 설명하는가
- local disk/cache의 위험을 이해하는가
- stateless가 downstream 병목을 없애는 것은 아니라는 점을 말하는가
