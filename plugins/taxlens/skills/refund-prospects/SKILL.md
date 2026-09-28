---
name: refund-prospects
description: 택스렌즈(TaxLens) MCP로 지역·단지·조합에서 취득세 환급 가망 대상(정비사업 조합·산업단지 입주자·오피스텔 임대 법인)을 찾는다. "부평구 환급 가망 대상 찾아줘", "이 재개발 조합 취득세 환급 여지", "남동산단 감면 누락 후보", "강서구 오피스텔 임대 법인 감면" 같은 질문에 쓴다. 절차 전문은 서버의 taxlens_get_guide가 주고, 이 스킬은 그것을 먼저 부르게 한다.
---

# 환급 가망 탐색 (refund-prospects)

## 먼저 할 일 — 반드시

**탐색 도구를 부르기 전에 `taxlens_get_guide(topic="refund-prospects")`를 부르고, 돌려받은 절차를 그대로 따른다.**
유형 라우팅, 기한 판단, 후보 확인 순서, 필요 자료 체크리스트, 금액 문구 규약이 모두 그 응답에 있다.

- 같은 대화에서 이미 읽었으면 다시 부르지 않는다.
- 가이드 내용을 파일로 저장하거나 요약본을 다른 곳에 남기지 않는다. 필요하면 다시 부른다.
- `taxlens_get_guide`가 도구 목록에 없으면 연결·권한 문제다. `taxlens_whoami`로 확인하고 사용자에게 알린다.
  가이드 없이 진행해야 하면 탐색 도구 응답의 안내 문구(금액 가정·기한·필요 자료)를 빠짐없이 옮긴다.

## 도구

| 순서 | 도구 |
|---|---|
| 0 절차 | `taxlens_get_guide(topic="refund-prospects")` |
| 1 탐색 | `taxlens_find_renewal_prospects` · `taxlens_find_industrial_complex_prospects` · `taxlens_find_officetel_rental_prospects` |
| 2 확인 | `taxlens_get_parcel` → `taxlens_get_ownership_timeline` (정비사업은 `taxlens_get_renewal_project`) |
| 3 근거 | `taxlens_search_precedents` → `taxlens_get_precedent`, 조문은 legal-kb `get_article` |

## 인용 원칙

- 금액은 도구가 준 **추정 범위(하한·상한)와 가정 문구**를 함께 옮긴다. 확정액·청구 가능액으로 바꾸지 않는다.
- 필지는 주소 + PNU, 판례는 사건번호 + 결정일 + 기관. 도구가 돌려주지 않은 PNU·금액·사건번호를 쓰지 않는다.
- 개인 소유자·조합원 이름을 추정하지 않는다.
- 답의 마지막 줄: 「추정치입니다. 실제 환급 여부는 신고서·감면신청서 등 자료를 받아 확정해야 합니다.」
