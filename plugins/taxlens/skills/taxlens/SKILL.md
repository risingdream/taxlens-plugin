---
name: taxlens
description: 택스렌즈(TaxLens) MCP로 부동산 물건·필지·과세표준 판단 규칙·부동산 세무 판례·소유권 변동을 조회해 근거로 답한다. "강남구 법인 소유 토지 찾아줘", "이 PNU 필지 검토", "취득세 과세표준에 ○○ 비용이 들어가나", "재건축 취득세 심판례" 같은 질문에 쓴다. 지역·단지의 환급 가망 대상 찾기는 refund-prospects 스킬이 맡는다. 절차 전문은 서버의 taxlens_get_guide가 준다. 법령 조문 원문은 legal-kb와 함께 쓴다.
---

# 택스렌즈 조회 (taxlens)

`taxlens` MCP는 택스렌즈 DB를 **읽기만** 한다(예외: `taxlens_lookup_owner`는 인터넷등기소를 조회해 저장한다).
답변 문장은 이 스킬을 읽는 모델이 쓴다.

## 먼저 할 일

- 조회 흐름·인자 요령·결과 해석은 서버가 준다. 처음 택스렌즈 질문을 받으면 `taxlens_get_guide(topic="taxlens")`를 부르고 따른다.
  인자·응답 필드 세부가 필요하면 `topic="taxlens-tools"`.
- **환급 가망 질문**(지역·단지·조합의 환급 대상, 감면 누락 후보)은 `refund-prospects` 스킬 — `taxlens_get_guide(topic="refund-prospects")`부터.
- 가이드 내용을 파일로 저장하지 않는다. 같은 대화에서 읽었으면 다시 부르지 않는다.
- 도구가 목록에 없으면 추측하지 말고 `taxlens_whoami`로 확인해 사용자에게 알린다.

## 흐름 — 호출 순서

```
물건 탐색   taxlens_search_properties(region, …) → taxlens_get_parcel(pnu)
필지 검토   taxlens_get_parcel(pnu) → taxlens_get_ownership_timeline(pnu)
소유자      taxlens_lookup_owner(pnu)  — 사용자가 고른 필지만, 분당 5회
과세표준   taxlens_list_tax_base_rules(tax_type, …) → taxlens_search_precedents / taxlens_get_precedent
판례 근거   taxlens_search_precedents(q, tax_type?) → taxlens_get_precedent(case_no) → legal-kb get_article
소유권 변동 taxlens_search_ownership_changes(region, date_from, date_to, …) → taxlens_get_ownership_timeline(pnu)
환급 가망   refund-prospects 스킬
```

- `region`은 시군구를 우선한다. 넓은 조회는 30초에 끊긴다 — 지역·기간을 좁혀 다시 부른다.
- 페이지를 넘겨 전량을 모으지 않는다. 첫 줄의 `N건 중 M건`을 그대로 전한다.

## legal-kb와 함께

택스렌즈에는 **법령 조문 원문이 없다.** 조문은 legal-kb `get_article(law_name, article_no, as_of)`,
택스렌즈에 없는 판례·결정례는 legal-kb `get_decision(case_no)`로 본다. 취득·과세 시점이 있으면 같은 날짜를 `as_of`로 넘긴다.
legal-kb가 없으면 조문을 기억으로 채우지 않고 「조문 원문은 확인하지 못했다」고 쓴다.

## 인용 형식

- **필지**: 주소 + PNU. 예) 서울특별시 강남구 역삼동 123-4 (PNU 1168010100101230004)
- **금액**: 원 단위, 천 단위 쉼표 또는 억·만원 환산을 함께. 추정 금액에는 「추정」을 붙인다.
- **판례·심판례**: 사건번호 + 결정일 + 기관, 가능하면 도구가 준 원 화면 링크.
- **과세표준 규칙**: 축 — 규칙 이름 + 판단(포함·제외·조건부) + 원 화면 링크.
- **조문**: 「법령명」 제N조 제M항 + 시행일(legal-kb 인용 형식).
- 출처를 밝힌다: 「택스렌즈 조회 기준(YYYY-MM-DD)」. 택스렌즈 값과 조문 원문을 한 문장에 섞지 않는다.

## 하지 않을 것

1. **개인 소유자 이름을 추정·복원하지 않는다.** `owner_name_withheld=true`면 이름이 비어 온다. 다른 출처로 채우지 않는다.
2. 도구가 돌려주지 않은 PNU·금액·사건번호·조문 번호를 쓰지 않는다. 못 찾았으면 못 찾았다고 쓴다.
3. 추정 금액을 확정처럼 쓰지 않는다. 세무 대리가 아니다.
4. 표 20행만 보고 「전체가 그렇다」고 일반화하지 않는다. `total`·`truncated`를 함께 전한다.

## 오류가 났을 때

| 응답 | 뜻과 대처 |
|---|---|
| 401 / 인증 필요 | 로그인이 끝나지 않았거나 만료됐다. Claude Code는 `/mcp` → `taxlens` → 인증 |
| 403 `mcp_disabled` | 조직의 MCP 연결이 꺼져 있다. 택스렌즈 운영자에게 조직 스위치를 요청한다 |
| 403 `mcp_forbidden` | 역할(열람 전용 `VIEWER` 등)이 MCP를 쓸 수 없다. 조직 관리자에게 역할 변경을 요청한다 |
| 403 `account_disabled` | 계정 또는 조직이 비활성이다 |
| 429 요청이 너무 많습니다 | 사용자 분당 60회·조직 분당 300회 상한. `retry_after_seconds` 뒤 다시 |
| 지역 후보 목록 | 지역 이름이 여러 곳과 겹친다. 후보의 이름 또는 코드로 다시 |
| 조회가 30초를 넘었습니다 | 시도→시군구, 기간 축소, 조건 추가 |
| 내부 오류(참조: …) | 서버 오류다. 참조 번호를 사용자에게 그대로 전한다 |

## 슬래시 커맨드

`/taxlens:물건` · `/taxlens:필지` · `/taxlens:판례` · `/taxlens:가망`(환급 가망 탐색).
