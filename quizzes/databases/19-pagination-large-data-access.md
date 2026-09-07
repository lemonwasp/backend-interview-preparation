# 19. Pagination / Large Data Access — Quiz

1. OFFSET Pagination이 깊은 페이지에서 느려질 수 있는 이유는 무엇인가요?
2. Keyset Pagination의 기본 아이디어를 설명하세요.
3. `created_at`만 Cursor로 쓰면 왜 중복/누락이 생길 수 있나요?
4. Stable Ordering을 위해 unique tie-breaker가 필요한 이유는 무엇인가요?
5. OFFSET Pagination이 여전히 좋은 선택일 수 있는 상황은 언제인가요?
6. Keyset Pagination의 단점은 무엇인가요?
7. 대량 Row를 한 번에 Application Memory로 가져오면 어떤 문제가 생기나요?
8. `SELECT *`가 대용량 조회에서 불리한 이유를 설명하세요.
9. Exact `COUNT(*)`가 꼭 필요한지 제품 요구사항과 함께 판단해야 하는 이유는 무엇인가요?
10. 변화가 많은 Feed에서 OFFSET이 페이지 중복/누락을 만들 수 있는 과정을 설명하세요.
11. `(tenant_id, created_at, id)` 같은 Composite Index가 Keyset Query에 어떻게 도움을 줄 수 있나요?
12. Pagination Query를 최적화할 때 EXPLAIN에서 무엇을 확인하겠습니까?
13. Batch Job이 500만 Row를 처리해야 합니다. 어떤 방식으로 읽겠습니까?
14. Keyset Pagination과 Cursor Pagination은 어떤 관계인가요?
15. 60초 안에 OFFSET vs Keyset Pagination을 설명하세요.

상태: **답변 대기**
