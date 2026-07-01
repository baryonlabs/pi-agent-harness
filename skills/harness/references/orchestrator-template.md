# 오케스트레이션 프롬프트 템플릿 (pi)

pi에서 오케스트레이터는 **프롬프트 템플릿**(`프로젝트/.pi/prompts/{name}.md`)으로 구현한다. 사용자가 `/{name}`으로 호출하면, 메인 에이전트가 번들된 `subagent` 도구를 single/parallel/chain으로 호출하며 워크플로우를 실행한다. 워크플로우가 복잡해 단계적 정보 공개가 필요하면, 얇은 프롬프트 + 상세 절차 스킬(`.pi/skills/{domain}-orchestrator/`)을 함께 둔다.

**모든 오케스트레이션 프롬프트의 공통 규칙:**
- `subagent` 도구 호출 시 **항상 `agentScope: "both"`를 명시** — 프로젝트 `.pi/agents/*.md`가 발견되게 한다. (기본값 `user`는 `~/.pi/agent/agents/`만 로드)
- 위임 모드는 작업 의존성에 맞춰 single/parallel/chain 중 선택. 복합이면 한 프롬프트가 여러 호출을 순서대로 지시.
- 큰 산출물은 `_workspace/`에 파일로 저장(parallel 출력은 task당 50KB 캡).

실행 모드별로 3가지 템플릿을 제공한다:
- **템플릿 A: 팬아웃/팬인 (parallel + 메인 통합)** — 가장 흔한 패턴
- **템플릿 B: 파이프라인 (chain)** — 순차 의존
- **템플릿 C: 하이브리드** — 모드 혼합

---

## 템플릿 A: 팬아웃/팬인 (parallel + 메인 통합)

독립적 자료를 병렬 수집한 뒤 메인 에이전트가 통합한다. pi엔 "팀 합의"가 없으므로 **메인이 통합자**다.

```markdown
---
description: "{도메인} 워크플로우 — 병렬 수집 후 통합. {초기 실행 키워드}. 후속 작업: {도메인} 결과 수정, 부분 재실행, 업데이트, 보완, 다시 실행, 이전 결과 개선 요청 시에도 이 프롬프트를 사용."
argument-hint: "<주제>"
---
{도메인} 자료를 수집·통합하여 {최종 산출물}을 생성하라. `subagent` 도구를 `agentScope: "both"`로 사용한다.

## Phase 0: 컨텍스트 확인 (후속 작업 지원)
1. `_workspace/` 존재 여부 확인
2. 실행 모드 결정:
   - 미존재 → 초기 실행. `_workspace/` 생성 후 Phase 1
   - 존재 + 부분 수정 요청 → 부분 재실행. 해당 에이전트만 재위임, 수정 대상 산출물만 덮어쓰기
   - 존재 + 새 입력 → 새 실행. 기존 `_workspace/`를 `_workspace_{YYYYMMDD_HHMMSS}/`로 이동 후 재생성

## Phase 1: 준비
- "$@"를 분석하여 조사 범위 파악
- 입력을 `_workspace/00_input.md`에 저장

## Phase 2: 병렬 수집
subagent 도구를 parallel 모드로 호출 (agentScope: "both"):
  tasks: [
    { agent: "{agent-a}", task: "$@ 에 대해 {영역A}를 조사하고 _workspace/02_{agent-a}.md에 저장. 핵심 요약만 반환." },
    { agent: "{agent-b}", task: "$@ 에 대해 {영역B}를 조사하고 _workspace/02_{agent-b}.md에 저장. 핵심 요약만 반환." },
    { agent: "{agent-c}", task: "$@ 에 대해 {영역C}를 조사하고 _workspace/02_{agent-c}.md에 저장. 핵심 요약만 반환." }
  ]
> parallel은 최대 8 task, 동시 4개. 출력 50KB 캡이므로 큰 결과는 파일로.

## Phase 3: 통합 (메인 에이전트)
1. `_workspace/02_*` 산출물을 모두 read
2. 상충 정보는 삭제하지 않고 출처 병기
3. 종합하여 `{output-path}/{filename}` 생성
> 통합 품질이 중요하면, single로 reviewer 에이전트에게 통합본 검수를 한 번 더 위임.

## Phase 4: 정리
- `_workspace/` 보존 (사후 검증·감사 추적용)
- 결과 요약 보고

## 에러 핸들링
- 에이전트 1개 실패: 1회 재위임. 재실패 시 누락 명시하고 진행
- 과반 실패: 사용자에게 알리고 진행 여부 확인
- 부분 결과로 통합 가능하면 진행, 보고서에 미수집 영역 명시

## 테스트 시나리오
- 정상: $@ 입력 → 3개 병렬 수집 → 통합 → {output} 생성
- 에러: {agent-b} 실패 → 1회 재위임 실패 → {agent-a},{agent-c} 결과로 통합, "{영역B} 일부 미수집" 명시
```

---

## 템플릿 B: 파이프라인 (chain)

순차 의존 작업. `{previous}`로 단계 간 출력 전달.

```markdown
---
description: "{도메인} 파이프라인 — {단계1}→{단계2}→{단계3}. {키워드}. 후속 작업 키워드 포함."
argument-hint: "<입력>"
---
"$@"에 대해 {도메인} 파이프라인을 실행하라. subagent 도구를 chain 모드로 사용한다 (agentScope: "both").

## Phase 0: 컨텍스트 확인
(템플릿 A와 동일 — `_workspace/` 존재 여부 분기)

## Phase 1: 체인 실행
subagent 도구를 chain 모드로 호출 (agentScope: "both"):
  chain: [
    { agent: "{scout}", task: "$@ 에 관련된 코드/자료를 찾아 압축된 컨텍스트로 반환" },
    { agent: "{planner}", task: "다음 컨텍스트로 '$@'의 실행 계획을 작성하라: {previous}" },
    { agent: "{worker}", task: "다음 계획을 구현하라: {previous}" }
  ]
> chain은 첫 실패 단계에서 멈춘다. 각 단계는 산출물을 `_workspace/{n}_{agent}.md`에도 저장하도록 task에 지시.

## Phase 2: 정리
- 최종 단계 출력을 `{output-path}`에 저장
- `_workspace/` 보존, 결과 요약 보고

## 에러 핸들링
- chain 중간 실패: 메인이 어느 단계가 실패했는지 보고. 부분 산출물(`_workspace/`)로 복구 가능하면 해당 단계부터 재실행
- 무한 재작업 방지: 생성-검증 루프는 chain 길이로 고정(예: worker→reviewer→worker 3단계)

## 테스트 시나리오
- 정상: $@ → scout → planner → worker → {output}
- 에러: planner 단계 실패 → 메인이 scout 결과로 직접 계획 초안 작성 후 worker 재개
```

---

## 템플릿 C: 하이브리드 (모드 혼합)

단계마다 다른 위임 모드를 사용한다. 한 프롬프트가 parallel → 메인 통합 → chain 검증을 순서대로 지시.

```markdown
---
description: "{도메인} 하이브리드 워크플로우. {키워드}. 후속 작업 키워드 포함."
argument-hint: "<입력>"
---
"$@"에 대해 다음 하이브리드 워크플로우를 실행하라. subagent 도구를 agentScope: "both"로 사용한다.

## Phase 1: 병렬 자료 수집 (parallel)
subagent parallel: [{agent-a}, {agent-b}, {agent-c}] — 각자 _workspace/01_{agent}.md에 저장

## Phase 2: 통합 (메인)
- 메인이 _workspace/01_* 를 read하여 통합 초안 _workspace/02_draft.md 작성

## Phase 3: 생성-검증 (chain)
subagent chain: [
  { agent: "{editor}", task: "_workspace/02_draft.md를 다듬어라" },
  { agent: "{qa-inspector}", task: "다음 결과를 검증하고 이슈를 리포트하라: {previous}" },
  { agent: "{editor}", task: "다음 검증 이슈를 반영해 최종본을 작성하라: {previous}" }
]

## Phase 4: 정리
- 최종본을 {output-path}에 저장, _workspace/ 보존
```

**하이브리드 전환 규칙:**
- parallel → 통합: 메인이 모든 parallel 결과를 read한 뒤 다음 단계 결정
- 통합 → chain: 통합 초안 파일 경로를 chain 첫 task에 명시
- 단계 간 대용량 데이터는 항상 `_workspace/` 파일로 (반환값 캡 회피)

---

## 작성 원칙

1. **위임 모드를 명시** — 각 Phase가 single/parallel/chain 중 무엇인지 task 구성으로 분명히
2. **agentScope: "both" 필수** — 빠지면 프로젝트 `.pi/agents/`가 발견되지 않아 위임 실패
3. **에이전트 이름이 실제 정의와 일치** — `.pi/agents/{name}.md`의 `name`과 동일해야 호출됨
4. **파일 경로는 `_workspace/` 기준 명확하게** — 상대 경로 혼동 금지
5. **단계 간 의존성 명시** — chain은 `{previous}`, 단계 간은 파일 경로
6. **에러 핸들링은 현실적으로** — "모든 것이 성공한다"고 가정하지 않음. chain 중단·parallel 부분 실패 대비
7. **테스트 시나리오 필수** — 정상 1 + 에러 1 이상

## description 작성 시 후속 작업 키워드

오케스트레이션 프롬프트 description은 초기 실행 키워드만으로는 부족하다. 다음 후속 표현을 반드시 포함하라:

- 재실행/다시 실행/업데이트/수정/보완
- "{도메인}의 {부분}만 다시"
- "이전 결과 기반으로", "결과 개선"
- 도메인 관련 일상적 요청 (예: 런치 전략 하네스라면 "런치", "홍보", "트렌딩" 등)

후속 키워드가 없으면 첫 실행 후 프롬프트가 사실상 죽은 코드가 된다.

## 얇은 프롬프트 + 오케스트레이터 스킬 (복잡한 워크플로우)

워크플로우 절차가 길어 프롬프트에 다 담기 어려우면, 프롬프트는 얇게 두고 상세를 스킬로 분리한다:

```markdown
---
description: "{도메인} 워크플로우 실행. {키워드}. 후속 작업 키워드 포함."
---
{domain} 작업을 수행한다. 먼저 `/skill:{domain}-orchestrator`를 read로 로드하여
전체 절차·에이전트 구성·데이터 흐름을 파악한 뒤, 그 지침에 따라 subagent 도구로 실행하라. ($@)
```

오케스트레이터 스킬(`.pi/skills/{domain}-orchestrator/SKILL.md`)에 에이전트 구성표·Phase별 위임 모드·데이터 흐름·에러 핸들링·테스트 시나리오를 담는다. 이렇게 하면 프롬프트는 트리거만, 상세는 단계적 정보 공개로 관리된다.

## 실제 오케스트레이션 참고

팬아웃/팬인 기본 구조: Phase 0(컨텍스트 확인) → parallel 수집 → 메인 read+통합 → 정리.
`references/team-examples.md`의 리서치 팀 예시를 참조.
