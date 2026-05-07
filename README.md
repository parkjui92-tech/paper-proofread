# paper-proofread

한국어 학술 논문 교정교열을 위한 [Claude Code](https://claude.com/claude-code) 스킬입니다.

논문 원고(`.docx` / `.pdf` / `.md` / `.hwp`)를 입력하면 청크 단위로 정밀 점검하여 **수정 원고**와 **교정교열표**를 출력합니다.

## 주요 기능

- **표기 교정**: 맞춤법, 띄어쓰기, 외래어, 문장부호
- **문장 교열**: 주술 호응, 이중피동, 번역투, 문장 길이, 중복 표현
- **논리·구조**: 문단 논리, 용어 일관성, 표/그림 번호
- **인용·참고문헌**: 본문 인용과 참고문헌 목록 대조, 서지정보 검증
- **심각도 3단계**: 🔴 error / 🟡 suggest / 🔵 style — strictness 옵션으로 출력 범위 조절

## 설치

Claude Code의 사용자 스킬 디렉터리(`~/.claude/skills/`)에 설치합니다.

### 방법 A — git clone (권장)

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/parkjui92-tech/paper-proofread.git
```

### 방법 B — `.skill` 번들 다운로드

```bash
mkdir -p ~/.claude/skills/paper-proofread
cd ~/.claude/skills/paper-proofread
curl -L -o paper-proofread.skill \
  https://github.com/parkjui92-tech/paper-proofread/raw/main/paper-proofread.skill
unzip paper-proofread.skill && rm paper-proofread.skill
```

설치 후 Claude Code를 재시작하면 자동으로 인식됩니다.

## 사용법

Claude Code에서 다음과 같이 자연어로 요청하면 스킬이 자동 트리거됩니다.

```
이 논문 교정교열해줘
첨부한 .docx 파일을 투고 전 검토해줘
원고 다듬어줘 (보수적으로)
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

예: "보수적으로 교정해줘" → `strictness: conservative` (🔴만 출력)

## 작동 방식

```
문서 읽기 → 청크 분할 → [청크별 점검 → 기록] × N → 통합 → 출력
```

전체 문서를 한 번에 처리하지 않고 약 2,000자 단위로 잘라 순차 점검하기 때문에, 긴 논문도 컨텍스트 한계 없이 정밀하게 다룰 수 있습니다. 청크 간에는 약어 정의·용어 표기·인용 목록을 추적하여 문서 전체의 일관성을 확보합니다.

자세한 내부 동작은 [SKILL.md](SKILL.md)를 참고하세요.

## 라이선스

자유롭게 사용·수정·배포 가능합니다.
