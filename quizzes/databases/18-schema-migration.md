# 18. Schema Migration — Quiz

1. 운영 환경의 Schema Migration이 단순 DDL 실행 이상인 이유는 무엇인가요?
2. Expand → Migrate → Contract를 설명하세요.
3. 구버전 App Instance가 남아 있는데 Column을 즉시 삭제하면 어떤 문제가 생기나요?
4. 큰 Table의 ALTER TABLE이 위험한 이유를 설명하세요.
5. 큰 Index 생성이 운영 Traffic에 줄 수 있는 영향은 무엇인가요?
6. 수천만 Row Backfill을 한 Transaction으로 처리하면 어떤 문제가 생길 수 있나요?
7. Backfill을 Batch로 나누는 이유는 무엇인가요?
8. 새 Column을 안전하게 NOT NULL로 바꾸는 흐름을 설명하세요.
9. Dual Write가 partial failure를 만들 수 있는 이유는 무엇인가요?
10. Migration rollback SQL이 있어도 안전한 배포라고 단정할 수 없는 이유는 무엇인가요?
11. Replica Lag이 Migration 중 증가할 수 있는 이유는 무엇인가요?
12. Migration 작업의 abort 기준으로 어떤 지표를 볼 수 있나요?
13. 실패 후 재실행 가능한 Migration은 어떤 특성을 가져야 하나요?
14. Zero-downtime Migration을 설계할 때 App과 DB 버전 호환성을 어떻게 생각해야 하나요?
15. 60초 안에 안전한 Schema Migration 원칙을 설명하세요.

상태: **답변 대기**
