# pi-agent-harness

pi(코딩 에이전트)용 **팀 아키텍처 팩토리**. 도메인 한 문장을 pi 에이전트 정의(`.pi/agents/`), 스킬(`.pi/skills/`), 오케스트레이션 프롬프트(`.pi/prompts/`)로 변환하는 메타 스킬 패키지.

## 하네스: 메타 스킬 자체

**목표:** 임의 도메인에 대해 pi 멀티에이전트 하네스를 생성·진화시킨다.

**트리거:** "하네스 구성해줘", "build a harness for this project", "pi 에이전트 팀 만들어줘" 등의 요청은 `harness` 스킬(`skills/harness/SKILL.md`)을 사용한다. 단순 질문은 직접 응답.

**핵심 구조:**
- `skills/harness/SKILL.md` — 7단계 워크플로우 (현황 감사 → 도메인 분석 → 아키텍처 설계 → 에이전트/스킬/프롬프트 생성 → 검증 → 진화)
- `skills/harness/references/` — 패턴·템플릿·예시·작성/테스트/QA 가이드 (조건부 로드)
- `extensions/subagent/` — 번들된 위임 도구(single/parallel/chain). pi엔 빌트인 서브에이전트가 없어 이 확장이 협업을 가능하게 함

**pi 매핑 핵심:** 협업 = `subagent` 도구 위임(single/parallel/chain) + `_workspace/` 파일 핸드오프. 오케스트레이션 프롬프트는 항상 `agentScope: "both"`를 명시해 프로젝트 `.pi/agents/`를 발견하게 한다. 자세한 매핑은 `skills/harness/references/agent-design-patterns.md` 참조.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-06-30 | Claude Code → pi 전용 포팅 | 전체 | pi 런타임 지원 |
