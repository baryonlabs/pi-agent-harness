# Agent Team Examples (pi)

pi 위임 모드(single/parallel/chain)로 구현한 하네스 예시 모음. 모든 협업은 메인 에이전트가 `subagent` 도구로 위임하고 `_workspace/` 파일로 결과를 주고받는다. pi엔 실시간 팀 메시징이 없으므로 **메인 에이전트가 통합·조정자** 역할을 맡는다.

---

## 예시 1: 리서치 팀 (parallel + 메인 통합)

### 패턴: 팬아웃/팬인 · 모드: parallel

```
[메인/오케스트레이터]
    ├── subagent parallel(4개 조사 위임)
    ├── _workspace/02_*.md 수집 (read)
    └── 종합 보고서 생성
```

### 에이전트 구성

| 에이전트 | model | tools | 역할 | 출력 |
|---------|-------|-------|------|------|
| official-researcher | claude-haiku-4-5 | read, grep, find, ls, bash | 공식 문서/블로그 | 02_official.md |
| media-researcher | claude-haiku-4-5 | read, grep, find, ls, bash | 미디어/투자 | 02_media.md |
| community-researcher | claude-haiku-4-5 | read, grep, find, ls, bash | 커뮤니티/SNS | 02_community.md |
| background-researcher | claude-sonnet-4-5 | read, grep, find, ls, bash | 배경/경쟁/학술 | 02_background.md |
| (메인 = 통합자) | — | — | 종합 보고서 | 종합보고서.md |

> 조사 에이전트는 정찰 성격이라 haiku로 충분, 배경 분석은 추론이 더 필요해 sonnet. 각 에이전트는 `.pi/agents/{name}.md`로 정의하고 역할·조사 범위·출력 경로를 본문에 명시.

### 오케스트레이션 프롬프트 흐름

```
Phase 0: _workspace/ 존재 확인 → 초기/새/부분 재실행 분기
Phase 1: "$@"(주제) 분석, _workspace/ 생성
Phase 2: subagent parallel (agentScope: "both"):
           [official, media, community, background]
           각자 02_{name}.md에 저장, 핵심 요약만 반환
Phase 3: 메인이 02_* 4개를 read → 종합
         상충 정보는 출처 병기 → 종합보고서.md
Phase 4: _workspace/ 보존, 결과 보고
```

### Claude 팀 → pi 매핑 메모

Claude 버전은 4명의 팀원이 `SendMessage`로 발견을 실시간 공유했다(예: media가 찾은 투자 뉴스를 background에 전달). pi엔 이 채널이 없으므로 **메인이 1차 parallel 결과를 읽고, 교차가 필요하면 2차 위임의 task에 끼워 넣는다**: 예) `subagent single({ agent: "background-researcher", task: "다음 투자 뉴스를 반영해 경쟁 환경을 보강하라: {media 결과 요약}" })`.

---

## 예시 2: SF 소설 집필 팀 (하이브리드)

### 패턴: 파이프라인 + 팬아웃 · 모드: parallel → 메인 통합 → single → chain

```
Phase 1 (parallel): worldbuilder + character-designer + plot-architect
Phase 2 (메인): 3개 설정을 read하여 일관성 정리
Phase 3 (single): prose-stylist 집필
Phase 4 (chain): science-consultant → continuity-manager → prose-stylist(수정)
```

### 에이전트 구성

| 에이전트 | model | 역할 | 스킬 |
|---------|-------|------|------|
| worldbuilder | claude-opus-4-7 | 세계관 구축 | world-setting |
| character-designer | claude-sonnet-4-5 | 캐릭터 설계 | character-profile |
| plot-architect | claude-opus-4-7 | 플롯 구조 | outline |
| prose-stylist | claude-opus-4-7 | 문체 편집 + 집필 | write-scene, review-chapter |
| science-consultant | claude-sonnet-4-5 | 과학 검증 | science-check |
| continuity-manager | claude-sonnet-4-5 | 일관성 검증 | consistency-check |

### 에이전트 파일 전문 예시: `.pi/agents/worldbuilder.md`

```markdown
---
name: worldbuilder
description: SF 소설의 세계관(물리 법칙, 사회 구조, 기술 수준, 역사)을 설계한다
model: claude-opus-4-7
---

당신은 SF 소설의 세계관 설계 전문가입니다. 과학적 사실에 기반하되 상상력을 확장하여, 이야기가 펼쳐질 세계의 물리적·사회적·기술적 토대를 구축합니다.

## 핵심 역할
1. 세계의 물리 법칙과 기술 수준 정의
2. 사회 구조, 정치 체계, 경제 시스템 설계
3. 역사적 맥락과 현재 갈등 구조 수립
4. 장소별 환경과 분위기 묘사

## 작업 원칙
- 내적 일관성 최우선 — 설정 간 모순이 없어야 한다
- "만약 이 기술이 있다면?" 연쇄 질문으로 세계의 파급 효과를 추론
- 이야기에 봉사하는 세계관 — 플롯을 방해하는 과도한 설정은 지양

## 입력/출력 프로토콜
- 입력: 사용자의 세계관 컨셉(task), 이전 산출물이 있으면 _workspace/에서 읽음
- 출력: `_workspace/01_worldbuilder_setting.md`
- 형식: 마크다운, 섹션별 (물리/사회/기술/역사/장소)
- 최종 반환: 핵심 설정 요약 (캐릭터·플롯 담당이 참조할 사회 구조·갈등 구조 중심)

## 사용 스킬
- 작업 전 `/skill:world-setting`을 read로 로드하여 설계 체크리스트를 따른다

## 재호출 지침
- _workspace/01_worldbuilder_setting.md가 존재하면 읽고, 피드백 반영분만 수정

## 에러 핸들링
- 컨셉이 모호하면 3가지 방향을 제안하고 선택 요청
- 과학적 오류 지적을 받으면 대안을 함께 제시
```

### 워크플로우 상세 (pi)

```
Phase 1: subagent parallel (agentScope: "both"):
           [worldbuilder, character-designer, plot-architect]
           각자 _workspace/01_{name}.md에 저장
Phase 2: 메인이 01_* 3개를 read → 설정 간 모순 점검, _workspace/01_bible.md로 정리
         (Claude의 "팀원 간 SendMessage 조율"을 메인의 일관성 통합으로 대체)
Phase 3: subagent single({ agent: "prose-stylist",
           task: "_workspace/01_bible.md를 바탕으로 {장면}을 집필하라" })
           → _workspace/02_prose_draft.md
Phase 4: subagent chain:
           [ {science-consultant, "draft의 과학 오류를 지적: 02_prose_draft.md"},
             {continuity-manager, "다음 지적에 일관성 이슈를 추가: {previous}"},
             {prose-stylist, "다음 리뷰를 반영해 최종 수정: {previous}"} ]
```

---

## 예시 3: 웹툰 제작 (생성-검증 chain)

### 패턴: 생성-검증 · 모드: chain

```
chain: [webtoon-artist 생성] → [webtoon-reviewer 검수] → [webtoon-artist 재생성]
```

### 에이전트 구성

| 에이전트 | model | 역할 | 스킬 |
|---------|-------|------|------|
| webtoon-artist | claude-opus-4-7 | 패널 이미지 생성 | generate-webtoon |
| webtoon-reviewer | claude-sonnet-4-5 | 품질 검수 | review-webtoon, fix-webtoon-panel |

### 에이전트 파일 전문 예시: `.pi/agents/webtoon-reviewer.md`

```markdown
---
name: webtoon-reviewer
description: 웹툰 패널의 구도·캐릭터 일관성·텍스트 가독성·연출을 검수한다
tools: read, grep, find, ls
model: claude-sonnet-4-5
---

당신은 웹툰 패널의 품질을 검수하는 전문가입니다. 시각적 완성도, 스토리 전달력, 캐릭터 일관성을 기준으로 패널을 평가합니다.

## 핵심 역할
1. 각 패널의 구도와 시각적 완성도 평가
2. 캐릭터 외형의 패널 간 일관성 검증
3. 말풍선 텍스트의 가독성과 배치 평가
4. 전체 에피소드의 연출 흐름과 페이싱 검토

## 작업 원칙
- PASS/FIX/REDO 3단계로 명확히 판정
- FIX는 부분 수정으로 해결 가능한 경우, REDO는 전면 재생성 필요
- 주관적 취향이 아닌 객관적 기준(일관성, 가독성, 구도)으로 판단

## 입력/출력 프로토콜
- 입력: `_workspace/panels/`의 패널 이미지들 (task로 경로 전달)
- 출력: `_workspace/review_report.md`
- 최종 반환: 패널별 판정 + 수정 지시 (다음 chain 단계의 artist가 {previous}로 받음)
- 형식:
  ## Panel {N}
  - 판정: PASS | FIX | REDO
  - 사유: [구체적 이유]
  - 수정 지시: [FIX/REDO인 경우 구체적 수정 방향]

## 에러 핸들링
- 이미지 로드 실패 시 해당 패널을 REDO로 판정
- 2회 재생성 후에도 REDO인 패널은 경고와 함께 PASS 처리
```

### 에러 핸들링 (재작업 제한)

chain은 길이로 재작업 횟수를 고정한다: `[artist, reviewer, artist]`는 1회 재작업. 더 필요하면 메인이 reviewer 리포트를 보고 추가 chain을 호출하되 최대 2회로 제한. 전체 패널의 50% 이상이 REDO면 사용자에게 프롬프트 수정 제안.

---

## 예시 4: 코드 리뷰 팀 (parallel + 메인 종합)

### 패턴: 팬아웃/팬인 · 모드: parallel

```
[메인] → subagent parallel:
    ├── security-reviewer: 보안 취약점 점검
    ├── performance-reviewer: 성능 영향 분석
    └── test-reviewer: 테스트 커버리지 검증
    → 메인이 3개 리포트를 read하여 교차 영역 이슈 종합
```

### Claude 팀 → pi 매핑 메모

Claude 버전은 리뷰어들이 `SendMessage`로 직접 교차 소통했다(security가 발견한 SQL 주입을 performance에게 알림 등). pi에서는 **메인이 종합 단계에서 교차 영역을 연결**한다: 3개 리포트를 읽고 "security의 SQL 주입 지점이 performance의 N+1 쿼리와 같은 파일" 같은 교차 이슈를 메인이 종합 리포트에 명시. 더 깊은 교차 분석이 필요하면 메인이 종합 결과를 가지고 single로 추가 위임.

### 에이전트 구성

| 에이전트 | model | tools |
|---------|-------|-------|
| security-reviewer | claude-sonnet-4-5 | read, grep, find, ls, bash (read-only) |
| performance-reviewer | claude-sonnet-4-5 | read, grep, find, ls, bash (read-only) |
| test-reviewer | claude-sonnet-4-5 | read, grep, find, ls, bash (read-only) |

> bash는 `git diff`/`git log` 등 read-only 명령만 쓰도록 각 에이전트 본문에 명시.

---

## 예시 5: 감독자 패턴 — 코드 마이그레이션 (메인의 동적 위임 루프)

### 패턴: 감독자 · 모드: 메인이 parallel을 반복 호출

```
[메인/감독자] → 파일 목록 분석 → 배치 분할
    반복:
      subagent parallel([migrator A배치, migrator B배치, migrator C배치])
      결과 수집 → 남은 파일 있으면 다음 배치 위임
    → 모든 배치 완료 → 통합 테스트
```

### 메인의 동적 분배 로직

pi엔 공유 작업 목록(TaskCreate)이 없으므로 **메인이 직접 루프**를 돈다:

```
1. 전체 대상 파일 목록 수집 (메인이 grep/find)
2. 복잡도 추정 (파일 크기, import 수)으로 배치 분할 (배치당 ≤8 파일)
3. subagent parallel로 한 배치(최대 8개, 동시 4개) 위임
   각 migrator는 결과를 _workspace/migrated/{file}.md에 기록하고 성공/실패 반환
4. 메인이 결과 수집:
   - 성공 파일 → 완료 목록에 추가
   - 실패 파일 → 다음 배치에 재투입 (1회) 또는 누락 명시
5. 남은 파일이 있으면 3으로 (다음 배치)
6. 전부 완료 → 메인이 통합 테스트 실행 (bash)
```

진행 상태는 메인의 컨텍스트 또는 `_workspace/progress.md`로 추적한다. Claude의 "워커 자체 작업 claim"은 **메인의 배치 분배 루프**로 대체된다.

---

## 산출물 패턴 요약

### 에이전트 정의 파일
위치: `프로젝트/.pi/agents/{agent-name}.md`
frontmatter: `name`, `description` (필수), `tools`, `model` (선택)
본문 = 위임된 pi 프로세스의 시스템 프롬프트. 필수: 핵심 역할, 작업 원칙, 입출력 프로토콜, 재호출 지침, 에러 핸들링.

### 스킬 파일 구조
위치: `프로젝트/.pi/skills/{skill-name}/SKILL.md` (프로젝트) 또는 `~/.pi/agent/skills/{skill-name}/SKILL.md` (글로벌)

### 오케스트레이션 프롬프트
위치: `프로젝트/.pi/prompts/{name}.md` → 사용자가 `/{name}`으로 호출
메인 에이전트가 `subagent` 도구를 single/parallel/chain으로 호출하도록 지시. **항상 `agentScope: "both"` 명시.**
템플릿: `references/orchestrator-template.md` 참조.
