# 택스렌즈 플러그인

택스렌즈(TaxLens)의 부동산 세무 데이터 — 물건·필지·환급 가망 탐색·과세표준 판단 규칙·판례 — 를
원격 MCP 서버(`https://lens.taxdesk.kr/mcp`)로 Claude 에 붙인다. 등기소 소유자 조회 하나를 빼면 읽기 전용이다.
도구는 택스렌즈 화면과 같은 조회를 한다. 답변 문장은 호출한 모델이 쓴다.

## 설치

```bash
claude plugin marketplace add risingdream/taxlens-plugin
claude plugin install taxlens@taxlens-plugin
```

Claude Desktop 은 설정 → 플러그인 → 추가 → 저장소에서 추가 → `risingdream/taxlens-plugin`.

설치 직후 `/mcp` 에서 `taxlens` 를 고르면 브라우저가 열린다. 택스렌즈 계정으로 로그인하고
「연결 허용」을 누르면 끝이다. 키를 복사해 붙여넣는 단계는 없다.

연결을 확인하려면 「택스렌즈 연결 확인해줘」라고 묻는다 — `taxlens_whoami` 가 계정·조직·쓸 수 있는 도구를 돌려준다.

### 연결이 안 될 때

| 증상 | 원인 | 할 일 |
|---|---|---|
| 로그인은 되는데 허용 뒤 403 `mcp_disabled` | 조직의 MCP 연결이 꺼져 있다 | 택스렌즈 운영자에게 조직 연결을 요청한다 |
| 403 `mcp_forbidden` | 열람 전용 역할이다 | 조직 관리자에게 역할 변경을 요청한다 |
| 401 이 반복된다 | 토큰 만료·폐기 | `/mcp` → `taxlens` → 다시 인증 |

## 도구

| 도구 | 쓰임 |
|---|---|
| `taxlens_whoami` | 연결 계정·조직·역할과 쓸 수 있는 도구 |
| `taxlens_get_guide` | 도구 호출 순서·결과 해석·인용 규약 전문(환급 가망 절차 포함). 스킬이 먼저 부른다 |
| `taxlens_search_properties` | 물건 탐색기 목록. 지역(시도·시군구) 필수 |
| `taxlens_get_parcel` | PNU 한 필지 상세 |
| `taxlens_list_tax_base_rules` | 과세표준 위키의 확정 판단 규칙 |
| `taxlens_search_precedents` | 부동산 세무 판례·심판례·유권해석 검색 |
| `taxlens_get_precedent` | 사건 한 건의 요지·연결 규칙·전문 |
| `taxlens_search_ownership_changes` | 지역·기간의 소유권 변동 사건, 법인 취득 후 양도 패턴 |
| `taxlens_get_ownership_timeline` | 필지 한 건의 소유권 변동 이력 |
| `taxlens_lookup_owner` | 필지 한 건의 등기부 소유자(법인명) 인터넷등기소 조회. 분당 5회 |
| `taxlens_find_renewal_prospects` | 정비사업 조합 신축 단지의 취득세 환급 가망(조합원·조합 원시취득, 재개발 감면) |
| `taxlens_get_renewal_project` | 정비사업 한 곳의 단계·보존 단지·조합원/조합 호실·기한 |
| `taxlens_find_industrial_complex_prospects` | 산업단지 입주자 감면 누락 가망, 필지 단위 |
| `taxlens_find_officetel_rental_prospects` | 오피스텔 임대사업자 최초 분양 감면 누락 가망, 건물·취득일 묶음 |

`taxlens_lookup_owner`만 외부 조회·저장을 하고 나머지는 읽기 전용이다. 택스렌즈 원장에는 소유자 이름이 없어,
법인명은 등기소 조회를 거친 필지만 보인다(나머지는 「미조회」).
개인 소유자 이름은 싣지 않는다(국공유·법인·종중·종교단체만 이름 공개, 등기 대표 명의가 개인이면 비공개).
사용자 분당 60회, 조직 분당 300회까지 부를 수 있다.

## 스킬 — 도구 사용법이 같이 온다

스킬 `taxlens`·`refund-prospects`는 얇다. 도구 이름·호출 순서·인용 원칙만 싣고, 절차 전문(환급 가망 유형 라우팅·
기한 판단·필요 자료 체크리스트·금액 문구 규약 등)은 **로그인한 연결에서 `taxlens_get_guide`가 서버에서 읽어 준다.**
스킬은 질문이 들어오면 그 도구를 먼저 부르게 한다. 가이드는 로컬 파일로 내려받지 않는다.

- 흐름 — 물건 탐색 · 필지 검토 · 환급 가망 탐색 · 과세표준 판단 · 판례 근거 확인에서 어떤 도구를 어떤 순서로 부를지
- 조회 범위 — 지역은 시군구로, 기간을 넣고, 페이지를 넘겨 전량을 모으지 않는다
- 추정 환급액 — 가망 탐색의 본세 추정 범위를 확정액처럼 쓰지 않는다
- 인용 형식 — 주소+PNU, 사건번호+결정일+원 화면 링크, 과세표준 규칙 축—이름+판단
- 개인정보 — 비공개 소유자 이름을 추정하지 않는다

## legal-kb 와 함께 쓴다

택스렌즈에는 **법령 조문 원문이 없다.** 환급 근거·판례가 해석한 조문의 항·호·단서는
[legal-kb 플러그인](https://github.com/risingdream/legal-kb-plugin)이 확인한다. 함께 설치하길 권한다.

```bash
claude plugin marketplace add risingdream/legal-kb-plugin
claude plugin install legal-kb@legal-kb-plugin
```

legal-kb 가 없으면 스킬은 조문을 기억으로 채우지 않고 「조문 원문은 확인하지 못했다」고 쓴다.

## 슬래시 커맨드

| 커맨드 | 하는 일 |
|---|---|
| `/taxlens:물건 <지역> [조건]` | 물건 탐색. 예: `/taxlens:물건 강남구 법인 이전 2025-01-01~2025-06-30` |
| `/taxlens:필지 <PNU 또는 주소>` | 필지 검토 |
| `/taxlens:판례 <사건번호 또는 검색어>` | 판례 전문과 연결된 과세표준 규칙 |
| `/taxlens:가망 <지역> [유형] [단지·조합명]` | 환급 가망 탐색. 예: `/taxlens:가망 부평구`, `/taxlens:가망 남동구 산업단지` |

## 업데이트

```bash
claude plugin marketplace update taxlens-plugin
claude plugin update taxlens@taxlens-plugin
```

## claude.ai 웹 채팅

claude.ai 는 플러그인을 받지 않는다. 설정 → 커넥터 → 커스텀 커넥터 추가에서
이름 `택스렌즈`, URL `https://lens.taxdesk.kr/mcp` 로 등록하고 브라우저에서 로그인·연결 허용한다.
도구 14종은 같고, 스킬·슬래시 커맨드는 붙지 않는다. 절차는 서버 안내와 `taxlens_get_guide`로 받는다.

Claude Code 에서 플러그인 없이 서버만 붙이려면:

```bash
claude mcp add --transport http taxlens https://lens.taxdesk.kr/mcp
```
