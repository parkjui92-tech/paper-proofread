# paper-proofread

[![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)](CHANGELOG.md)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

한국어 학술논문 및 보고서 교정교열을 위한 [Claude Code](https://claude.com/claude-code) 스킬입니다.

논문 원고(`.docx` / `.pdf` / `.md` / `.hwp`)를 입력하면 청크 단위로 정밀 점검하여 **수정 원고**와 **교정교열표**를 출력합니다.

> **English**: A Claude Code skill for proofreading Korean academic manuscripts. It splits the document into ~2,000-character chunks, checks orthography / sentence grammar / logic / citations sequentially with a curated Korean style-rule reference (`references/rules_ko.md`), and produces a corrected manuscript plus an itemized correction table with 3-level severity. Optional reference-existence verification (DOI/URL) flags hallucinated citations in AI-assisted drafts.

## 주요 기능

- **표기 교정**: 맞춤법, 띄어쓰기, 외래어, 문장부호
- **문장 교열**: 주술 호응, 이중피동, 번역투, 문장 길이, 중복 표현
- **논리·구조**: 문단 논리, 용어 일관성, 표/그림 번호
- **인용·참고문헌**: 본문 인용과 참고문헌 목록 대조, 서지정보 검증
- **서지 실재 검증 (선택, v1.1)**: DOI·URL을 실제로 조회해 **환각 출처(지어낸 문헌)** 를 플래그 — AI가 관여한 원고의 투고 전 점검에 유용
- **심각도 3단계**: 🔴 error / 🟡 suggest / 🔵 style — strictness 옵션으로 출력 범위 조절

## 예시 출력 (교정교열표)

| # | 위치 | 심각도 | 유형 | 원문 | 수정안 | 사유 |
|---|------|--------|------|------|--------|------|
| 1 | §2.1/C3 | 🔴 | 맞춤법 | 데이타 | 데이터 | 외래어 표기법 |
| 2 | §2.1/C3 | 🔴 | 이중피동 | 분석되어지고 | 분석되고 | 이중피동 제거 |
| 3 | §3.2/C5 | 🟡 | 번역투 | ~에 있어서 | ~에서 | 불필요한 번역투 |
| 4 | 참고문헌 | 🔴 | 인용 불일치 | (김철수, 2021) | — | 본문 인용이 목록에 없음 |

## 설치

Claude Code의 사용자 스킬 디렉터리(`~/.claude/skills/`)에 설치합니다.

### 방법 A — git clone (권장)

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/parkjui92/paper-proofread.git
```

### 방법 B — `.skill` 번들 다운로드

```bash
mkdir -p ~/.claude/skills/paper-proofread
cd ~/.claude/skills/paper-proofread
curl -L -o paper-proofread.skill \
  https://github.com/parkjui92/paper-proofread/raw/main/paper-proofread.skill
unzip paper-proofread.skill && rm paper-proofread.skill
```

설치 후 Claude Code를 재시작하면 자동으로 인식됩니다.

## 사용법

Claude Code에서 다음과 같이 자연어로 요청하면 스킬이 자동 트리거됩니다.

```
이 논문 교정교열해줘
첨부한 .docx 파일을 투고 전 검토해줘
원고 다듬어줘 (보수적으로)
참고문헌 실재 여부까지 검증해줘   ← 서지 실재 검증 켜기
```

다음 키워드 중 하나라도 포함되면 스킬이 활성화됩니다:
교정 · 교열 · 교정교열 · proofreading · 논문 검토 · 논문 수정 · 원고 다듬기 · 맞춤법 검사 · 투고 전 검토 · 문장 교열 · 학술 글쓰기 점검

## 옵션

| 파라미터 | 기본값 | 설명 |
|----------|--------|------|
| `chunk_size` | 2000자 | 청크 크기 (문단 경계에서 분할) |
| `citation_style` | APA 7th | 인용 형식 |
| `strictness` | moderate | conservative / moderate / aggressive |
| `focus` | full | full / surface / sentence / reference |
| `verify_references` | off | on이면 DOI·URL 실재 검증 (네트워크 필요) |

예: "보수적으로 교정해줘" → `strictness: conservative` (🔴만 출력)

## 요구 환경

- Claude Code(로컬) 또는 claude.ai — 양쪽 경로를 모두 지원합니다 (v1.1).
- 선택 의존성 (없으면 자동 폴백):
  - `.docx` 추출: `pandoc` → 없으면 `python-docx` → docx 스킬
  - `.hwp/.hwpx` 추출: [kordoc](https://github.com/chrisryugj/kordoc) MCP/CLI → 없으면 LibreOffice
  - `.pdf` 추출: pdf 스킬 또는 `pypdf`

## 작동 방식

```
문서 읽기 → 청크 분할 → [청크별 점검 → 기록] × N → 통합 → 출력
```

전체 문서를 한 번에 처리하지 않고 약 2,000자 단위로 잘라 순차 점검하기 때문에, 긴 논문도 컨텍스트 한계 없이 정밀하게 다룰 수 있습니다. 청크 간에는 약어 정의·용어 표기·인용 목록을 추적하여 문서 전체의 일관성을 확보합니다.

원고 본문은 외부로 전송되지 않습니다. 네트워크를 사용하는 것은 (옵션을 켠 경우의) 서지 실재 검증뿐이며, 그때도 서지정보(제목·저자·DOI)만 조회합니다.

자세한 내부 동작은 [SKILL.md](SKILL.md)를 참고하세요.

## 연구자용 스킬 시리즈

이 스킬은 "검증이 내장된 연구 워크플로" 시리즈의 첫 번째 도구입니다.

- **paper-proofread** (이 저장소) — 한국어 학술 원고 교정교열
- **fact-verify** — 조사 결과·브리핑의 출처 신뢰도 검증 (4-Tier 분류, 재인용 체인 감지, URL/DOI 실존 확인)

## 라이선스

[Apache License 2.0](LICENSE)
