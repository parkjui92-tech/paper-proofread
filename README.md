# paper-proofread

[![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

**한국어** · [English](README.en.md)

한국어 학술 원고(`.docx` · `.pdf` · `.md` · `.hwp`)를 약 2,000자 청크로 잘라 끝까지 훑고 **수정 원고**와 **교정교열표**를 내놓는 [Claude Code](https://claude.com/claude-code) 스킬입니다.
맞춤법 검사기가 보는 표기 층위 아래로 한 겹 더 들어갑니다 — 이중피동·번역투·주술 불일치 같은 **문장 층위**, 그리고 본문 인용과 참고문헌 목록이 맞물리는 **인용 층위**까지.

<!-- 데모 GIF 자리 -->

## 무엇을 점검하나

네 축입니다 — **표기**(맞춤법·띄어쓰기·외래어 표기·숫자 단위), **문장**(주술 호응·이중피동·번역투·중복 표현), **논리·구조**(문단 구조·용어 일관성·표 번호와 본문 참조), **인용·참고문헌**(본문 인용 ↔ 목록 대조·서지정보 완결성). 영어권 교정 도구에는 이중피동·번역투가 범주 자체로 없고, 국내 맞춤법 검사기는 표기 층위 위를 보지 않습니다.
선택 축이 하나 더 있습니다. **서지 실재 검증**은 DOI·URL을 실제로 열어 조회된 메타데이터를 목록과 대조합니다. AI가 초안이나 문헌 정리에 관여한 원고의 투고 전 점검에서 가장 값진 축입니다 — 그럴듯하지만 존재하지 않는 문헌은 서지 형식이 오히려 깔끔해서 눈으로는 걸러지지 않습니다.

| # | 위치 | 심각도 | 유형 | 원문 | 수정안 | 사유 |
|---|------|--------|------|------|--------|------|
| 1 | §2.1/C3 | 🔴 | 이중피동 | 분석되어지고 | 분석되고 | 이중피동 제거 |
| 2 | §3.2/C5 | 🟡 | 번역투 | ~에 있어서 | ~에서 | 불필요한 번역투 |
| 3 | 참고문헌 | 🔴 | 환각 출처 의심 | (목록 12번 문헌) | — | DOI 조회 실패 · 해당 권호에 논문 없음 |

형식 예시입니다. 지적마다 심각도 🔴 error(반드시 수정) / 🟡 suggest(권고) / 🔵 style(선택)이 붙고, 위치 칼럼의 `§2.1/C3`은 섹션명과 청크 번호라 원문에서 바로 찾아갈 수 있습니다. 이 **교정교열표**와 함께 **수정 원고**, 그리고 반복 오류 패턴·용어 불일치를 정리한 **요약 리포트**가 나옵니다.
→ [왜 만들었나·상세 사용법](docs/why.md)

## 설치

```bash
# A. git clone (권장 — 이후 git pull로 갱신)
mkdir -p ~/.claude/skills && cd ~/.claude/skills && git clone https://github.com/parkjui92/paper-proofread.git
# B. .skill 번들 (git 없이 파일만)
mkdir -p ~/.claude/skills/paper-proofread && cd ~/.claude/skills/paper-proofread
curl -L -o paper-proofread.skill https://github.com/parkjui92/paper-proofread/raw/main/paper-proofread.skill
unzip paper-proofread.skill && rm paper-proofread.skill
# 설치 후 Claude Code 재시작 → ~/.claude/skills/paper-proofread/SKILL.md 가 있으면 완료
```

## 쓰는 법

```
이 논문 투고 전에 교정교열해줘. 첨부한 .docx야   ← 네 축 전체 (논문 아닌 문서도 됩니다)
참고문헌 실재 여부까지 검증해줘                  ← 서지 실재 검증 on
맞춤법이랑 띄어쓰기만 빠르게 봐줘                ← 축 선택 · "보수적으로"면 🔴만
```

| 파라미터 | 기본값 | 설명 |
|---|---|---|
| `strictness` | moderate | `conservative`(🔴만) / `moderate`(🔴🟡) / `aggressive`(🔴🟡🔵) |
| `focus` | full | `full` / `surface`(표기) / `sentence`(문장) / `reference`(인용·참고문헌) |
| `verify_references` | off | on이면 DOI·URL 실재 검증 (네트워크 필요) |
| `chunk_size` · `citation_style` | 2000자 · APA 7th | 청크 크기(문단 경계에서 끊음) · 인용 형식 |

## 요구 환경·한계

- Claude Code(로컬)·claude.ai 모두 지원. `.docx`·`.pdf`·`.hwp` 추출은 의존성이 없으면 자동 폴백하고 어떤 경로를 썼는지 알립니다. 원고 본문은 외부로 전송되지 않습니다 — 네트워크를 쓰는 것은 켜 둔 경우의 서지 실재 검증뿐이고, 그때도 조회하는 것은 서지정보뿐입니다
- **놓치는 것도, 잘못 짚는 것도 있습니다.** 교정교열표는 최종 결정이 아니라 검토 목록입니다. 서지 실재 검증의 "미확인"(DOI 없는 국내 학술지·페이월)도 오류가 아니라 사람이 직접 봐야 할 목록입니다
- **내용은 건드리지 않습니다.** 논증·방법론 층위의 검수는 [socsci-paper-kit](https://github.com/parkjui92/socsci-paper-kit)의 검수 게이트 몫입니다
- [SKILL.md](SKILL.md) · [references/rules_ko.md](references/rules_ko.md)(번역투 패턴 14종 등 교정 규칙) · [CHANGELOG.md](CHANGELOG.md)

## 시리즈

"검증이 내장된 연구 워크플로" 시리즈의 하나입니다. 셋은 팀으로 문서를 끝까지 쓰는 킷이고, 하나는 강의자료를 만들고 화면에서 고치는 킷이며, 넷은 워크플로의 한 지점을 담당하는 단독 스킬입니다.

**에이전트 팀 킷** — 설계 → 검토 게이트 → 조사 → 집필 → 검수 → 변환을 팀으로 완주하는 플러그인

- [policy-research-kit](https://github.com/parkjui92/policy-research-kit) — 정책연구보고서
- [rnd-proposal-kit](https://github.com/parkjui92/rnd-proposal-kit) — 정부 R&D 제안서
- [socsci-paper-kit](https://github.com/parkjui92/socsci-paper-kit) — 사회과학 논문. **이 스킬을 동반 스킬로 씁니다** — 팀의 마무리 담당이 승인된 초안을 교정교열할 때 paper-proofread를 호출합니다

**제작·편집 킷** — 강의자료를 만들고 브라우저에 띄운 채로 고치는 플러그인

- [lecture-deck-kit](https://github.com/parkjui92/lecture-deck-kit) — 강의자료 HTML 덱 + 라이브 편집

**단독 스킬**

- [paper-proofread](https://github.com/parkjui92/paper-proofread) — 한국어 학술 교정교열 **(이 저장소)**
- [fact-verify](https://github.com/parkjui92/fact-verify) — 출처 검증 (4-Tier 분류, 재인용 체인 감지, URL·DOI 실존 확인)
- [form-tailor](https://github.com/parkjui92/form-tailor) — 기관 양식 맞춤
- [report-to-brief](https://github.com/parkjui92/report-to-brief) — 보고서 압축

## 라이선스

[MIT](LICENSE)
