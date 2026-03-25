# My Claude Code Skills & Agents

개인적으로 사용하는 Claude Code 글로벌 스킬과 에이전트를 관리하는 저장소입니다.

## 설치 방법

`~/.claude/` 디렉토리에 심볼릭 링크를 생성하여 사용합니다.

```bash
# 스킬 설치 (각 스킬별)
ln -s $(pwd)/.claude/skills/<skill-name> ~/.claude/skills/<skill-name>

# 에이전트 설치 (각 에이전트별)
ln -s $(pwd)/.claude/agents/<agent-name>.md ~/.claude/agents/<agent-name>.md
```

---

## Skills (19개)

Claude Code에서 `/skill-name` 으로 호출하는 슬래시 커맨드 스킬입니다.

### 개발 파이프라인

| 스킬 | 설명 | 호출 예시 |
|------|------|-----------|
| **[grill-me](.claude/skills/grill-me/)** | 아이디어 심문/검토 — 구현 전 설계의 모든 분기를 탐색 | `/grill-me` |
| **[write-prd](.claude/skills/write-prd/)** | PRD/요구사항 문서 작성 | `/write-prd` |
| **[prd-to-issues](.claude/skills/prd-to-issues/)** | PRD를 vertical slice 기반 실행 가능한 이슈로 분해 | `/prd-to-issues` |
| **[suggest-next](.claude/skills/suggest-next/)** | 개발 파이프라인 내비게이터 — 다음 단계 안내 | `/suggest-next` |

### 코드 분석 & 리뷰

| 스킬 | 설명 | 호출 예시 |
|------|------|-----------|
| **[find-in-code](.claude/skills/find-in-code/)** | 코드베이스 탐색/검색 — 절대경로:라인번호 형식 | `/find-in-code` |
| **[diff-review](.claude/skills/diff-review/)** | 코드 변경사항 before/after 비교 분석 | `/diff-review` |
| **[codebase-health](.claude/skills/codebase-health/)** | 코드베이스 구조 건강도 진단 | `/codebase-health` |
| **[my-review](.claude/skills/my-review/)** | 개인용 PR/코드 리뷰 도우미 | `/my-review` |

### 구현 지원

| 스킬 | 설명 | 호출 예시 |
|------|------|-----------|
| **[propose-fix](.claude/skills/propose-fix/)** | 구현 방안 비교 제안서 — 다중 방안 비교 + 사이드이펙트 분석 | `/propose-fix` |
| **[karpathy-guidelines](.claude/skills/karpathy-guidelines/)** | LLM 코딩 실수 방지 가이드라인 (Andrej Karpathy 기반) | 자동 트리거 |
| **[vercel-react-best-practices](.claude/skills/vercel-react-best-practices/)** | React/Next.js 성능 최적화 가이드 (Vercel Engineering) | 자동 트리거 |

### Git & Jira

| 스킬 | 설명 | 호출 예시 |
|------|------|-----------|
| **[create-pr](.claude/skills/create-pr/)** | 구조화된 PR 자동 생성 (Walkthrough + Mermaid 다이어그램) | `/create-pr` |
| **[jira-write](.claude/skills/jira-write/)** | Jira 티켓 생성 및 댓글 작성 | `/jira-write` |

### 프론트엔드 디자인

| 스킬 | 설명 | 호출 예시 |
|------|------|-----------|
| **[adapt](.claude/skills/adapt/)** | 다양한 화면 크기/디바이스/플랫폼에 맞게 디자인 적용 | `/adapt` |
| **[bolder](.claude/skills/bolder/)** | 안전하거나 밋밋한 디자인을 더 임팩트 있게 개선 | `/bolder` |
| **[clarify](.claude/skills/clarify/)** | UX 카피, 에러 메시지, 마이크로카피 개선 | `/clarify` |
| **[normalize](.claude/skills/normalize/)** | 디자인 시스템에 맞게 정규화 | `/normalize` |
| **[optimize](.claude/skills/optimize/)** | 인터페이스 성능 최적화 (로딩, 렌더링, 번들) | `/optimize` |
| **[polish](.claude/skills/polish/)** | 출시 전 최종 품질 점검 (정렬, 간격, 일관성) | `/polish` |

---

## Agents (7개)

Claude Code의 Agent 시스템으로 실행되는 자율 에이전트입니다.

| 에이전트 | 설명 | 모델 |
|----------|------|------|
| **[dev-cycle](.claude/agents/dev-cycle.md)** | PRD/이슈 → 구현 완료까지의 오케스트레이터 (계획→아키텍처 리뷰→TDD→코드 리뷰→문서) | Sonnet |
| **[investigate-issue](.claude/agents/investigate-issue.md)** | 이슈 조사 → 원인 분석 → 영향 범위 → 해결 계획 → 문서화 | - |
| **[plan-first](.claude/agents/plan-first.md)** | 코드 작성 전 구현 계획을 먼저 제시하고 승인 후 실행 | Sonnet |
| **[tdd](.claude/agents/tdd.md)** | Red→Green→Refactor TDD 사이클로 코드 구현 | Sonnet |
| **[senior-frontend-reviewer](.claude/agents/senior-frontend-reviewer.md)** | 프론트엔드 클린 코드 & 구현 품질 리뷰 | - |
| **[senior-frontend-architect](.claude/agents/senior-frontend-architect.md)** | 프론트엔드 아키텍처 검토 (SOLID, 선언형 패턴) | - |
| **[trace-flow](.claude/agents/trace-flow.md)** | End-to-End 코드 흐름 시각화 (시퀀스 다이어그램) | - |

---

## 파이프라인 흐름

```
① grill-me → ② write-prd → ③ prd-to-issues → ④ dev-cycle
   (심문)       (PRD 작성)     (이슈 분해)         (구현)
                                                   ├─ investigate-issue (조사)
                                                   ├─ plan-first (계획)
                                                   ├─ tdd (TDD 구현)
                                                   ├─ senior-frontend-reviewer (리뷰)
                                                   ├─ senior-frontend-architect (아키텍처)
                                                   └─ trace-flow (흐름 시각화)
```

---

## 디렉토리 구조

```
.
├── README.md
└── .claude/
    ├── skills/
    │   ├── adapt/SKILL.md
    │   ├── bolder/SKILL.md
    │   ├── clarify/SKILL.md
    │   ├── codebase-health/SKILL.md
    │   ├── create-pr/SKILL.md
    │   ├── diff-review/SKILL.md
    │   ├── find-in-code/SKILL.md
    │   ├── grill-me/SKILL.md
    │   ├── jira-write/SKILL.md
    │   ├── jira-write-workspace/evals/trigger-eval.json
    │   ├── karpathy-guidelines/SKILL.md
    │   ├── my-review/SKILL.md
    │   ├── normalize/SKILL.md
    │   ├── optimize/SKILL.md
    │   ├── polish/SKILL.md
    │   ├── prd-to-issues/SKILL.md
    │   ├── propose-fix/SKILL.md
    │   ├── suggest-next/SKILL.md
    │   ├── vercel-react-best-practices/
    │   │   ├── SKILL.md
    │   │   └── AGENTS.md
    │   └── write-prd/SKILL.md
    └── agents/
        ├── dev-cycle.md
        ├── investigate-issue.md
        ├── plan-first.md
        ├── senior-frontend-architect.md
        ├── senior-frontend-reviewer.md
        ├── tdd.md
        └── trace-flow.md
```
