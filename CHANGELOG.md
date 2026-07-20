# Changelog

## [Unreleased]

### Changed
- **라이선스 Apache-2.0 → MIT.** 이 저장소의 내용은 프롬프트·마크다운 자산이라 Apache-2.0의 핵심 이점인 특허 실시권 조항에 실익이 없고, §4(b)의 "변경 사실 고지" 의무가 이 저장소의 의도된 사용(각자 분야·기관에 맞게 고쳐 쓰기)과 오히려 충돌한다. Claude Code 스킬·플러그인 생태계의 사실상 표준도 MIT다. MIT는 Apache-2.0보다 제약이 적으므로 기존 이용자의 권리는 축소되지 않는다.
- README를 국문·영문 2종으로 전면 개편 — 제작 배경과 활용 시나리오 보강 (`README.md` / `README.en.md`)

## [1.1.0] - 2026-07-13

### Added
- **서지 실재 검증(선택)**: `verify_references: on` 시 Phase 3에서 DOI·URL을 실제 조회해 환각 출처(실존하지 않는 문헌)를 🔴로 플래그. AI가 관여한 원고의 투고 전 점검용
- LICENSE (Apache-2.0), CHANGELOG.md
- README: 버전·라이선스 배지, 교정교열표 예시, 요구 환경(폴백 의존성), 영문 요약, 연구자용 스킬 시리즈 안내
- 원고 보호 원칙 명문화: 원고 본문을 외부로 전송하지 않음(서지 검증 시 서지정보만 조회)

### Changed
- **환경 이식성**: 파일 경로가 claude.ai 컨테이너(`/mnt/user-data/uploads/`, `/home/claude/`) 전제로 고정돼 있던 것을 Claude Code 로컬/클라우드 겸용으로 수정 (작업 파일은 현재 작업 디렉터리의 `proofread_work/`)
- **추출 폴백 체계화**: `.docx`는 pandoc→python-docx→docx 스킬, `.hwp/.hwpx`는 kordoc MCP→kordoc CLI→LibreOffice 순으로 시도하고 사용한 방법을 보고. LibreOffice 경로의 표 평탄화 위험 경고 추가
- 요약 리포트에 서지 실재 검증 결과 항목 추가

## [1.0.0] - 2026-05 이전 (초기 공개)

- 청크 단위 순차 점검(기본 2,000자), 4축 점검(표기·문장·논리·인용), 심각도 3단계(🔴🟡🔵), strictness/focus 옵션
- 한국어 교정 규칙 레퍼런스 `references/rules_ko.md`
- `.skill` 번들 배포 방식
