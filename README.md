# taxlens-plugin

부동산 세무 인텔리전스 서비스 **택스렌즈(TaxLens)** 의 물건·필지·환급 가망 탐색·과세표준 판단 규칙·판례를
조회하는 원격 MCP 서버(`https://lens.taxdesk.kr/mcp`)의 에이전트 플러그인 마켓플레이스다.
답변 문장은 만들지 않는다. 호출한 모델이 근거를 받아 쓴다.

이 저장소에는 MCP 연결 설정과 도구 사용 절차(스킬·슬래시 커맨드)만 있다. 서버와 데이터는 공개하지 않는다.

## 설치

| 클라이언트 | 방법 |
|---|---|
| Claude Code | `claude plugin marketplace add risingdream/taxlens-plugin` 후 `claude plugin install taxlens@taxlens-plugin` |
| Claude Desktop (Code·Cowork) | 설정 → 플러그인 → 추가 → 저장소에서 추가 → `risingdream/taxlens-plugin` |
| claude.ai 웹 채팅 | 플러그인 대신 커넥터: 설정 → 커넥터 → 커스텀 커넥터 추가 → 이름 `택스렌즈`, URL `https://lens.taxdesk.kr/mcp` |

설치 뒤 첫 연결 때 브라우저 로그인·연결 허용 화면이 한 번 뜬다. Claude Code 는 `/mcp` → `taxlens` → 인증.
키를 복사해 붙여넣는 단계는 없다.

**MCP 연결이 켜진 조직의 택스렌즈 계정만 연결된다.** 로그인은 되는데 허용 뒤 막히면
택스렌즈 운영자에게 조직 연결을 요청한다.

법령 조문 원문은 [legal-kb-plugin](https://github.com/risingdream/legal-kb-plugin)과 함께 쓴다.

## 업데이트

```bash
claude plugin marketplace update taxlens-plugin
claude plugin update taxlens@taxlens-plugin
```

## 들어 있는 것

- 도구 14종 — `taxlens_whoami` · `taxlens_get_guide`(사용 절차 전문, 로그인 연결에서만) · `taxlens_search_properties` · `taxlens_get_parcel` ·
  `taxlens_list_tax_base_rules` · `taxlens_search_precedents` · `taxlens_get_precedent` ·
  `taxlens_search_ownership_changes` · `taxlens_get_ownership_timeline` · `taxlens_lookup_owner`(등기소 소유자 조회 — 유일하게 외부 조회·저장) ·
  `taxlens_find_renewal_prospects` · `taxlens_get_renewal_project` · `taxlens_find_industrial_complex_prospects` ·
  `taxlens_find_officetel_rental_prospects`(정비사업·산업단지·오피스텔 환급 가망 탐색)
- 스킬 `taxlens` — 흐름별 도구 호출 순서, 인용 형식, 개인 소유자 비공개 원칙, legal-kb 연계
- 스킬 `refund-prospects` — 환급 가망 질문이면 `taxlens_get_guide`로 절차를 먼저 받게 하는 얇은 스킬
- 커맨드 `/taxlens:물건` · `/taxlens:필지` · `/taxlens:판례` · `/taxlens:가망` (Claude Code·Desktop 전용)

자세한 내용은 [`plugins/taxlens/README.md`](plugins/taxlens/README.md).

> 이 저장소는 비공개 서비스 저장소의 `plugins/taxlens/` 에서 동기화된다. 여기에 직접 커밋하지 않는다.
