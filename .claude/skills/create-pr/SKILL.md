---
name: create-pr
description: 구조화된 PR 자동 생성 — diff 분석 후 Walkthrough 요약, 파일 그룹별 Changes 테이블, Mermaid 시퀀스 다이어그램(실행 흐름 시각화), 테스트 플랜을 포함한 PR을 생성합니다. git-convention 머지 전략 자동 적용. "PR 만들어줘", "PR 생성해줘", "develop에 PR", "main에 PR", "풀리퀘스트 생성", "PR 올려줘" 등 PR 생성 요청 시 반드시 이 스킬을 사용하세요.
user_invocable: true
---

# PR 생성 스킬

PR 생성 요청 시 아래 절차를 **순서대로** 실행합니다.

## STEP 1: 정보 수집

다음 명령을 **병렬**로 실행하여 정보를 수집합니다:

```bash
# 1. 현재 브랜치와 상태
git branch --show-current
git status

# 2. 타겟 브랜치 대비 커밋 히스토리
git log --oneline <target-branch>..HEAD

# 3. 타겟 브랜치 대비 변경 파일 통계
git diff <target-branch>...HEAD --stat

# 4. 타겟 브랜치 대비 전체 diff
git diff <target-branch>...HEAD

# 5. 리모트 브랜치 존재 여부
git branch -r | grep <current-branch>
```

**타겟 브랜치 결정 규칙:**
- 사용자가 명시한 경우 → 해당 브랜치
- 명시하지 않은 경우 → `develop` (기본값)
- `origin/<target>` 우선, 없으면 로컬 `<target>`

## STEP 2: 변경 분석

수집한 diff를 기반으로 아래 항목을 분석합니다:

1. **변경 유형 판별**: feat / fix / refactor / chore / mixed
2. **파일 그룹핑**: 관련 파일을 Cohort(응집 단위)로 묶기
3. **흐름 파악**: 사용자 액션 → 프론트엔드 → API → 백엔드 순서의 실행 흐름
4. **영향 범위**: 어떤 사용자/페이지/플랫폼에 영향이 있는지

## STEP 3: PR 본문 작성

아래 **정확한 템플릿**으로 PR body를 생성합니다:

```markdown
## Walkthrough

> **한줄 요약**: {변경의 핵심을 한 문장으로. 리뷰어가 3초 안에 맥락 파악 가능해야 함}

{1~3문장으로 상세 배경 설명. 왜 이 변경이 필요했는지, 어떤 비즈니스 문제를 해결하는지.
긴 문장은 줄바꿈하여 가독성 확보}

&nbsp;

## Changes

| 변경 그룹 | 파일 | 요약 |
|---|---|---|
| **{그룹 이름}** | `{파일명1}`, `{파일명2}` | {변경 요약} |
| **{그룹 이름}** | `{파일명}` | {변경 요약} |

&nbsp;

## Sequence Diagram

```mermaid
sequenceDiagram
    participant {한글alias}
    participant {한글alias} as {컴포넌트명}
    ...
```

&nbsp;

## Test Plan

- [ ] {테스트 항목 1}
- [ ] {테스트 항목 2}
- [ ] {기존 기능 미영향 확인}

---

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### 섹션 간격 규칙 (GitHub 렌더링)

GitHub 마크다운에서 섹션이 붙어 보이지 않도록, 각 `##` 섹션 사이에 `&nbsp;` (빈 줄 + non-breaking space + 빈 줄)을 삽입한다.
Walkthrough와 Changes 사이, Changes와 Sequence Diagram 사이, Sequence Diagram과 Test Plan 사이에 각각 넣는다.
Walkthrough 본문이 길면 문장 단위로 줄바꿈하여 GitHub에서 단락이 분리되도록 한다.

### 템플릿 작성 규칙

**Walkthrough:**
- `> **한줄 요약**:` blockquote로 핵심 한 문장 먼저 제시 — 리뷰어가 PR 목록에서 3초 안에 맥락을 잡을 수 있어야 함
- 그 아래 1~3문장으로 배경과 비즈니스 이유 설명
- 기술 용어보다 **"왜" 이 변경이 필요했는지**에 집중
- 예:
  > **한줄 요약**: 사용자 프로필 편집 페이지에 이미지 크롭 기능 추가
  >
  > 기존에는 원본 이미지가 그대로 업로드되어 용량/비율 문제가 빈번했습니다. 클라이언트에서 크롭 후 업로드하도록 변경합니다.

**Changes 테이블:**
- 3컬럼: `변경 그룹` | `파일` | `요약`
- 그룹 이름은 **볼드**, **짧고 직관적**으로 (예: "인증 로직", "UI 컴포넌트", "API 레이어")
- 파일은 **파일명만** 표시 (경로 생략). 동명 파일이 있을 때만 부모 디렉토리 1단계 포함
  - Good: `useAuth.ts`, `ProfileForm.tsx`
  - Bad: `src/features/auth/hooks/useAuth.ts`
- 요약은 **한국어**, 동사로 시작 (예: "이미지 크롭 후 업로드 로직 구현")
- 그룹은 최대 **5~6개** — 너무 많으면 리뷰어가 읽지 않음

**Sequence Diagram:**
- participant는 **최대 5~6개**. 너무 많으면 중간 레이어를 합치거나 생략
  - Bad: Controller, Service, Repository, Validator, Mapper 를 각각 분리 (8개)
  - Good: Backend(서비스+리포지토리), ExternalAPI 로 합치기 (5개)
- participant 이름은 **짧은 한글 alias** 권장 (예: `participant 프로필페이지 as ProfilePage`)
- `alt`/`else`로 핵심 분기만 표현 — 모든 에러 케이스를 넣지 말고 대표 흐름에 집중
- `Note`는 꼭 필요한 보충만 (재시도 횟수, 타임아웃 등)
- 단순 변경(스타일, 타입 추가 등)은 다이어그램 **생략**

**Test Plan:**
- 체크박스(`- [ ]`) 형식
- 플랫폼별 테스트 (Desktop/Mobile/iOS/AOS) 분리
- 기존 기능 미영향 확인 항목 필수 포함
- 최소 3개, 최대 8개

## STEP 4: PR 제목 결정

git-convention 스킬의 커밋 메시지 형식을 따릅니다:
```
[Jira-Ticket-ID] type(scope): 한국어 요약 (50자 이내)
```

- 브랜치명에서 이슈 ID 자동 추출 (예: `feature/PROJ-44-...` → `[PROJ-44]`)
- 커밋 히스토리에서 주요 type 판별
- 복수 type이면 가장 큰 비중의 type 사용

## STEP 5: 푸시 및 PR 생성

```bash
# 1. 리모트 브랜치가 없으면 푸시
git push -u origin <current-branch>

# 2. PR 생성 (HEREDOC으로 body 전달)
gh pr create --base <target-branch> --title "<제목>" --body "$(cat <<'EOF'
<본문>
EOF
)"
```

## STEP 6: 결과 보고

PR 생성 후 아래 형식으로 보고합니다:

```
PR 생성 완료:
- **URL**: {PR URL}
- **방향**: {source} → {target}
- **머지 전략**: {git-convention 기준 머지 전략}
```

## 머지 전략 참조 (git-convention)

| 머지 방향 | 전략 |
|----------|------|
| feature → develop | Merge Commit |
| hotfix → develop | Merge Commit |
| feature → release | Squash and Merge |
| qa → release | Squash and Merge |
| release → main | Merge Commit |
| hotfix → main | Squash and Merge |

## 주의사항

- PR 생성 **전에** 반드시 `pre-commit-review` + `senior-frontend-reviewer` 에이전트를 병렬 실행하여 코드 리뷰를 완료해야 합니다 (피드백 메모리 규칙)
- 리뷰 결과를 사용자에게 보여주고 **승인 후** PR 생성을 진행합니다
- unstaged/untracked 변경이 있으면 사용자에게 알리고 커밋 여부를 확인합니다
- `docs/` 등 untracked 디렉토리는 PR 범위에 포함하지 않습니다
