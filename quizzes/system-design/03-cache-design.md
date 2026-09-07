# 03. Cache Design — Interview Drills

상태: **답변 대기**

## Questions

1. Cache를 사용하는 핵심 목적은 무엇인가요?
2. Cache-Aside 패턴의 read/write 흐름을 설명하세요.
3. Write-Through와 Write-Behind의 차이와 trade-off는 무엇인가요?
4. TTL을 너무 짧게 또는 너무 길게 설정하면 어떤 문제가 생기나요?
5. Cache Hit Ratio만 높으면 좋은 시스템이라고 볼 수 없는 이유는 무엇인가요?
6. Cache Stampede가 무엇이며 어떻게 완화할 수 있나요?
7. Cache Penetration과 Cache Avalanche의 차이를 설명하세요.
8. Hot Key가 distributed cache에서도 문제가 되는 이유는 무엇인가요?
9. Local Cache와 Distributed Cache의 장단점을 비교하세요.
10. Cache Key에 tenant/user 정보를 잘못 포함하면 어떤 보안 문제가 생길 수 있나요?
11. Eviction Policy가 왜 중요한가요?
12. DB update 성공 후 cache invalidation 실패 시 어떻게 보완할 수 있나요?
13. CDN을 cache 관점에서 설명하세요.
14. Cache를 넣지 않는 편이 더 나은 경우를 말해보세요.

## Evaluation Criteria

- Cache를 단순 속도 기술이 아니라 downstream load reduction으로 설명하는가
- Cache-Aside와 consistency 문제를 함께 설명하는가
- Stampede/Penetration/Avalanche/Hot Key를 구분하는가
- Local vs Distributed trade-off를 설명하는가
- TTL/eviction/key design을 operational 관점에서 이해하는가
- Cache가 source of truth가 아니라는 점을 명확히 하는가
