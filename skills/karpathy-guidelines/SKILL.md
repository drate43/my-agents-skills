---
name: karpathy-guidelines
description: LLM 코딩 실수 방지 가이드라인 — 코드를 작성, 수정, 리팩토링할 때 과도한 엔지니어링과 스코프 크립을 방지합니다. "구현해줘", "만들어줘", "고쳐줘", "리팩토링해줘", "추가해줘", "변환해줘", "에러 바운더리 추가", "useEffect 리팩토링", "API route 만들어줘", "타입스크립트로 변환", "retry 로직 추가", "버그 수정해줘" 등 코드를 생성하거나 변경하는 모든 요청에서 반드시 이 스킬을 참조하세요. 코드 변경이 수반되는 작업이라면 규모에 관계없이 항상 적용합니다. 단순한 코드 검색이나 설명 요청에는 트리거하지 마세요.
license: MIT
---

# Karpathy Guidelines

Behavioral guidelines to reduce common LLM coding mistakes, derived from [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

## 한국어 핵심 요약

| 원칙 | 한 줄 요약 |
|------|-----------|
| 생각 먼저 | 가정을 명시하고, 불확실하면 멈추고 질문 |
| 단순하게 | 요청한 것만 구현. 추측 금지, 과잉 추상화 금지 |
| 수술적 변경 | 내가 바꾼 것만 손대고, 주변 코드 건드리지 않기 |
| 목표 기반 실행 | 성공 기준을 먼저 정의하고, 검증될 때까지 반복 |
