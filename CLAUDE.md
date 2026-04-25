# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> 7-Phase Context Engineering 파이프라인 — AI에게 보낼 컨텍스트를 잘 만드는 Claude Code 플러그인

## Project Nature

소스 코드 없음. **순수 마크다운** Claude Code 플러그인:
- `.claude-plugin/plugin.json` — 플러그인 메타데이터 (v3.1.0)
- `skills/context-engineering/SKILL.md` — 오케스트레이터 (전체 7-Phase 파이프라인)
- `skills/context-engineering/references/` — 10개 템플릿 및 참조 문서
- `skills/gather/SKILL.md` — Phase 1-3 (문제 정의 → 수집 → 선택)
- `skills/build/SKILL.md` — Phase 4-5 (구조화 → 압축)
- `skills/compose/SKILL.md` — Phase 6 (실행 지시 생성)
- `skills/verify/SKILL.md` — Phase 7 (최종 검증)

## Pipeline

```
Phase 1 →[G1]→ Phase 2 →[G2]→ Phase 3 →[G3]→
Phase 4 →[G4]→ Phase 5 →[G5]→ Phase 6 →[G6]→ [출력]
                                                  ↓ (G6 실패)
                                                Phase 7
```

| 서브 스킬 | Phase | 역할 |
|----------|-------|------|
| `gather` | 1-3 | 문제 정의 → 후보 수집 → 선택 |
| `build` | 4-5 | 구조화 → 압축 |
| `compose` | 6 | 실행 지시 생성 |
| `verify` | 7 | 최종 검증 |

Phase 6은 Phase 1 success criteria 신호에 따라 출력 형식을 자동 결정:
- **Format A**: 실행 지시 (Role + Task + Constraints + Output Format)
- **Format B**: KB 엔트리 (frontmatter + Key Facts/Constraints/Decisions/Notes)
- **Format C**: 프로젝트 산출물 (CLAUDE.md + spec.md + implementation plan)

## Skill Modification Guidelines

1. **Phase 일관성**: gather → build → compose → verify 흐름 유지
2. **템플릿 동기화**: `references/` 파일 10개와 SKILL.md 인라인 구조를 항상 일치시킬 것
3. **게이트 형식**: `"Phase N 결과: {요약}\n 미충족 항목: {내용}\n 수정하거나 승인 후 진행해주세요."` 준수
4. **게이트 동작**: 기준 명확히 충족 → 자동 통과, 모호한 경우 → 사용자 확인 요청
5. **Compaction Survival**: 각 SKILL.md의 핵심 행동 계약을 **첫 5,000 토큰** 내에 배치. 보조 정보는 그 뒤에 둘 것 — 컨텍스트 압축 시 뒷부분이 잘림

## Verification

스킬 수정 후 실제 프로젝트에서 `/context-engineering @<path>` 실행하여 Phase 1 질의가 올바르게 시작되는지 확인.
