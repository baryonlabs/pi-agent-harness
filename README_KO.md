<p align="center">
  <img src="harness_banner.png" alt="Harness Banner" width="600">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Version-2.0.0-brightgreen.svg" alt="Version">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License"></a>
  <img src="https://img.shields.io/badge/pi-Package-purple.svg" alt="pi Package">
  <img src="https://img.shields.io/badge/Patterns-6_Architectures-orange.svg" alt="6 Architecture Patterns">
  <img src="https://img.shields.io/badge/Modes-single%20%7C%20parallel%20%7C%20chain-green.svg" alt="Delegation Modes">
  <a href="https://github.com/revfactory/harness/stargazers"><img src="https://img.shields.io/github/stars/revfactory/harness?style=social" alt="GitHub Stars"></a>
</p>

<p align="center">
  <a href="#카테고리--harness는-어디에-서-있나요"><img src="https://img.shields.io/badge/Layer-L3%20Meta--Factory-orange" alt="Layer"></a>
  <a href="#카테고리--harness는-어디에-서-있나요"><img src="https://img.shields.io/badge/Sub--layer-Team--Architecture%20Factory-teal" alt="Sub-layer"></a>
  <a href="#"><img src="https://img.shields.io/badge/README-EN%20%7C%20KO%20%7C%20JA-lightgrey" alt="i18n"></a>
</p>

# pi-agent-harness — pi를 위한 팀 아키텍처 팩토리

[English](README.md) | **한국어** | [日本語](README_JA.md)

> **pi-agent-harness는 [pi](https://github.com/earendil-works/pi)(미니멀 터미널 코딩 에이전트)를 위한 팀 아키텍처 팩토리입니다.** **"하네스 구성해줘"** (한국어) · **"build a harness for this project"** (English) · **"ハーネスを構成して"** (日本語) 한 문장으로, 도메인 설명을 pi 에이전트 정의(`.pi/agents/`), 그들이 쓸 스킬(`.pi/skills/`), 이들을 구동하는 오케스트레이션 프롬프트(`.pi/prompts/`)로 변환합니다 — 사전 정의된 6가지 팀 아키텍처 패턴 중 하나를 골라서요.

> **Claude Code에서 포팅됨.** 이 패키지는 원본 [Harness](https://github.com/revfactory/harness)(Claude Code 플러그인)를 **pi 전용으로 포팅**한 것입니다. 동일한 팩토리를 pi의 프리미티브(`.pi/agents/`, `.pi/skills/`, `.pi/prompts/`, 번들된 `subagent` 확장)에 매핑했습니다. pi는 의도적으로 **빌트인 서브에이전트·MCP·플랜모드·투두를 제공하지 않기** 때문입니다.

## 개요

pi는 의도적으로 미니멀합니다: 모델에게 네 가지 도구(`read`, `write`, `edit`, `bash`)만 주고, 나머지는 스킬·프롬프트 템플릿·확장·패키지로 추가합니다. 이 하네스는 그 확장성을 활용해 복잡한 작업을 전문 에이전트로 분해·조율합니다. "하네스 구성해줘"라고 말하면 다음을 생성합니다:

- **에이전트 정의** (`.pi/agents/*.md`) — `tools`/`model` frontmatter를 가진 전문가 페르소나. 각자 격리된 `pi` 프로세스로 실행됨
- **스킬** (`.pi/skills/*/SKILL.md`) — Agent Skills 표준 절차적 지식
- **오케스트레이션 프롬프트** (`.pi/prompts/*.md`) — `subagent` 도구를 `single`/`parallel`/`chain` 모드로 구동하는 `/커맨드`

번들된 **`subagent` 확장**(`extensions/subagent/`)이 pi에 없는 위임 도구를 공급하므로, 생성된 하네스는 추가 설치 없이 바로 동작합니다.

## 카테고리 — Harness는 어디에 서 있나요

Harness는 **L3 메타 팩토리** 계층 — 다른 하네스를 *생성*하는 계층 — 에 위치합니다. L3 안에서 우리는 특정 하위 계층을 고릅니다: **팀 아키텍처 팩토리**. 이 패키지는 **pi 런타임**을 타깃으로 합니다.

| 계층 | 하는 일 | 이웃 |
|------|---------|------|
| **L3 — 메타 팩토리 / 팀 아키텍처 팩토리** (우리) | 도메인 한 문장 → pi 에이전트 팀 + 스킬 + 오케스트레이션 프롬프트, 6가지 팀 패턴으로 | — |
| L3 — 메타 팩토리 / 런타임 구성 팩토리 | 결정론적·반복 가능한 런타임 구성 | [coleam00/Archon](https://github.com/coleam00/Archon) |
| L3 — 메타 팩토리 / Claude Code 원본 | 동일 개념, Claude Code 런타임 | [revfactory/harness](https://github.com/revfactory/harness) |
| L2 — 크로스 하네스 워크플로우 | 여러 하네스에 걸쳐 스킬/규칙/훅 표준화 | [affaan-m/ECC](https://github.com/affaan-m/everything-claude-code) |

## 핵심 기능

- **에이전트 설계** — 6가지 아키텍처 패턴: 파이프라인, 팬아웃/팬인, 전문가 풀, 생성-검증, 감독자, 계층적 위임 — pi의 `single`/`parallel`/`chain` 위임 모드에 매핑
- **스킬 생성** — 효율적 컨텍스트 관리를 위한 Progressive Disclosure를 갖춘 Agent Skills 표준 스킬 자동 생성
- **오케스트레이션** — `{previous}` 체이닝, `_workspace/` 파일 핸드오프, 에러 핸들링을 갖춘 `subagent` 도구 프롬프트
- **모델 티어링** — 에이전트별 모델 선택 (정찰=`claude-haiku-4-5`, 작업=`claude-sonnet-4-5`, 고난도 추론=`claude-opus-4-7`)
- **검증** — 트리거 검증, 드라이런 테스트, `subagent` 도구를 통한 스킬 유무 비교 테스트

## 워크플로우

```
Phase 1: 도메인 분석
    ↓
Phase 2: 아키텍처 설계 (single / parallel / chain 위임)
    ↓
Phase 3: 에이전트 정의 생성 (.pi/agents/)
    ↓
Phase 4: 스킬 생성 (.pi/skills/)
    ↓
Phase 5: 오케스트레이션 프롬프트 (.pi/prompts/) + AGENTS.md 포인터
    ↓
Phase 6: 검증 및 테스트
    ↓
Phase 7: 하네스 진화 (피드백 기반)
```

## 설치

pi 패키지로 설치하면 번들된 `subagent` 확장과 `harness` 스킬이 자동 로드됩니다:

```bash
# git에서
pi install npm:@baryonlabs/pi-agent-harness

# git
pi install git:github.com/baryonlabs/pi-agent-harness

# 또는 로컬 체크아웃
pi install /absolute/path/to/pi-agent-harness

# 프로젝트 로컬 설치 (.pi/settings.json으로 팀과 공유 가능)
pi install -l /absolute/path/to/pi-agent-harness
```

설치 없이 시험:

```bash
pi -e /absolute/path/to/pi-agent-harness
```

그런 다음 pi 안에서 트리거합니다:

```
하네스 구성해줘
Build a harness for this project
```

## 패키지 구조

```
pi-agent-harness/
├── package.json                    # pi 매니페스트 (extensions + skills)
├── extensions/
│   └── subagent/                   # 번들된 위임 도구 (single/parallel/chain)
│       ├── index.ts
│       ├── agents.ts
│       └── README.md
├── skills/
│   └── harness/
│       ├── SKILL.md                # 메인 스킬 (7단계 워크플로우)
│       └── references/
│           ├── agent-design-patterns.md   # 6가지 패턴 + Claude→pi 매핑
│           ├── orchestrator-template.md   # 오케스트레이션 프롬프트 템플릿
│           ├── team-examples.md           # 5가지 실전 구성
│           ├── skill-writing-guide.md     # 스킬 작성 가이드
│           ├── skill-testing-guide.md     # 테스트·평가 방법론
│           └── qa-agent-guide.md          # QA 에이전트 통합 가이드
└── README.md
```

## 사용법

### 위임 모드

pi에는 빌트인 에이전트 팀이나 실시간 메시징이 없습니다. 협업은 메인 에이전트가 번들된 `subagent` 도구로 위임하고 `_workspace/` 파일로 결과를 교환하는 방식으로 구현됩니다. 오케스트레이션 프롬프트는 반드시 `agentScope: "both"`를 넘겨 프로젝트 레벨 `.pi/agents/`가 발견되게 해야 합니다.

| 모드 | 도구 파라미터 | 용도 |
|------|--------------|------|
| **single** | `{ agent, task }` | 단발 격리 작업, 전문가 풀 선택 |
| **parallel** | `{ tasks: [...] }` (최대 8, 동시 4) | 팬아웃/팬인 독립 작업; 메인이 통합 |
| **chain** | `{ chain: [...] }` + `{previous}` | 순차 파이프라인, 생성-검증 루프 |

<p align="center">
  <img src="harness_team.png" alt="Harness Agent Team" width="500">
</p>

### 아키텍처 패턴 → pi 모드

| 패턴 | pi 매핑 |
|------|---------|
| 파이프라인 | `chain` |
| 팬아웃/팬인 | `parallel` → 메인이 통합 |
| 전문가 풀 | `single` (선택적) |
| 생성-검증 | `chain` (worker → reviewer → worker) |
| 감독자 | 메인 에이전트의 동적 `parallel` 루프 |
| 계층적 위임 | 2단계 위임 |

## 산출물

Harness가 프로젝트에 생성하는 파일:

```
your-project/
├── .pi/
│   ├── agents/          # 에이전트 정의 (name, description, tools, model)
│   │   ├── analyst.md
│   │   ├── builder.md
│   │   └── qa-inspector.md
│   ├── skills/          # 스킬 파일
│   │   ├── analyze/
│   │   │   └── SKILL.md
│   │   └── build/
│   │       ├── SKILL.md
│   │       └── references/
│   └── prompts/         # 오케스트레이션 프롬프트 (/커맨드)
│       └── build-feature.md
└── AGENTS.md            # 하네스 포인터 (트리거 규칙 + 변경 이력)
```

## 사용 사례 — 이런 프롬프트를 써보세요

설치 후 pi 안에서 아무거나 트리거하세요:

**딥 리서치** — `딥 리서치용 하네스를 만들어줘: 병렬 에이전트가 공식·미디어·커뮤니티 관점에서 주제를 조사하고, 메인 에이전트가 교차 검증하여 보고서를 작성.`

**웹사이트 개발** — `풀스택 웹사이트 개발 하네스를 chain으로 만들어줘: 설계 → 프론트엔드(React/Next.js) → 백엔드 API → QA 테스트.`

**코드 리뷰** — `종합 코드 리뷰 하네스를 만들어줘: 병렬 에이전트가 아키텍처·보안·성능을 점검한 뒤 하나의 리포트로 통합.`

**웹툰 제작** — `웹툰 에피소드 하네스를 만들어줘: artist 에이전트가 패널을 생성하고 reviewer 에이전트가 스타일 일관성을 강제하는 생성-검증 체인.`

**마케팅 캠페인** — `마케팅 캠페인 하네스를 만들어줘: 시장 조사, 광고 카피 작성, 비주얼 컨셉 설계, A/B 테스트 계획을 반복 검토와 함께.`

## 요구 사항

- [pi 코딩 에이전트](https://github.com/earendil-works/pi) 설치 (`npm install -g --ignore-scripts @earendil-works/pi-coding-agent`)
- pi에 인증된 모델 프로바이더 (`/login` 또는 `ANTHROPIC_API_KEY` 같은 API 키)
- 번들된 `subagent` 확장 (이 패키지 설치 시 자동) — 추가 설정 불필요

> **보안 참고:** `subagent` 도구는 `agentScope: "both"`/`"project"`로 호출될 때만 프로젝트 로컬 에이전트(`.pi/agents/*.md`)를 실행하며, 인터랙티브 모드에서는 확인을 요청합니다. 신뢰하는 저장소에서만 활성화하세요. `extensions/subagent/README.md` 참조.

## Harness로 만든 것들 (Claude Code 원본)

원본 Claude Code 하네스는 통제된 A/B 실험으로 검증되었고 대규모 카탈로그를 배포했습니다. 해당 산출물은 Claude Code를 타깃으로 하지만, 방법론은 이 pi 포팅에도 그대로 적용됩니다.

- **[revfactory/harness-100](https://github.com/revfactory/harness-100)** — 10개 도메인의 프로덕션급 에이전트 팀 하네스 100종 (EN + KO).
- **[revfactory/claude-code-harness](https://github.com/revfactory/claude-code-harness)** — A/B 실험(n=15): 평균 품질 +60% (49.5 → 79.3), 15/15 승률, 분산 −32% (저자 측정, 제3자 재현 대기).

> 전체 논문: *Hwang, M. (2026). Harness: Structured Pre-Configuration for Enhancing LLM Code Agent Output Quality.*

## FAQ

<details>
<summary><b>Q1. 원본 Harness와 무엇이 다른가요?</b></summary>

**A.** 같은 팩토리, 다른 런타임입니다. 원본 [revfactory/harness](https://github.com/revfactory/harness)는 `.claude/agents/` + `.claude/skills/`를 생성하고 Claude Code의 네이티브 에이전트 팀(`TeamCreate`/`SendMessage`/`TaskCreate`)에 의존하는 Claude Code 플러그인입니다. 이 패키지는 **pi 전용 포팅**입니다: `.pi/agents/` + `.pi/skills/` + `.pi/prompts/`를 생성하고, 번들된 `subagent` 확장(`single`/`parallel`/`chain`) + `_workspace/` 파일 핸드오프로 협업을 구현합니다. pi에는 빌트인 서브에이전트나 실시간 팀 메시징이 없기 때문입니다.
</details>

<details>
<summary><b>Q2. pi에 서브에이전트가 없는데 에이전트 팀은 어떻게 동작하나요?</b></summary>

**A.** 이 패키지는 위임마다 별도 `pi` 프로세스(격리된 컨텍스트 윈도우)를 스폰하는 `subagent` 확장을 번들합니다. 메인 에이전트가 조정자 역할을 합니다: `parallel`로 팬아웃하고, `chain`으로 의존 단계를 엮으며(`{previous}` 전달), 결과를 스스로 통합합니다(pi엔 팀 합의 채널이 없음). 전체 Claude→pi 매핑은 `skills/harness/references/agent-design-patterns.md` 참조.
</details>

<details>
<summary><b>Q3. 기존 Claude Code 스킬을 재사용할 수 있나요?</b></summary>

**A.** 네. pi는 `settings.json`에 디렉토리를 추가하여 다른 하네스의 스킬을 로드할 수 있습니다 (예: `"skills": ["~/.claude/skills"]`). Agent Skills 표준 SKILL.md 파일은 Claude Code와 pi 사이에서 대체로 이식 가능합니다.
</details>

## 라이선스

Apache 2.0
