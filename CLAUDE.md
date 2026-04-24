# context-engineering

This file provides guidance to Claude Code when working with code in this repository.

> AI 도구로 복잡한 서비스를 개발할 때의 4단계 Context Engineering 워크플로우 Claude Code 플러그인

## 프로젝트 성격

소스 코드 없음. **순수 마크다운** Claude Code 플러그인:
- `plugin.json` — 플러그인 메타데이터
- `skills/context-engineering/SKILL.md` — 스킬 정의 (워크플로우 핵심)
- `skills/context-engineering/references/` — 각 Phase 산출물 템플릿

## 스킬 수정 시 주의사항

1. **변수 일관성**: `{OUTPUT_PATH}` 등 Step 0에서 정의한 변수만 이후 Phase에서 참조
2. **템플릿 동기화**: `references/` 템플릿과 SKILL.md 본문의 산출물 구조 일치 유지
3. **게이트 메시지 형식**: `"Phase {N} 완료 — \`{산출물}\` 초안을 저장했습니다. 검토 후 Phase {N+1} 진행할까요?"` 통일

## 검증

스킬 변경 후 실제 프로젝트에서 `/context-engineering @<경로>`를 실행해 Step 0 파라미터 수집이 정상인지 확인.
