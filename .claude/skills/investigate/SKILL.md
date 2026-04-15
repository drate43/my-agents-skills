---
name: investigate
description: 이슈/버그 코드 조사 — Jira 이슈 키·URL·키워드로 프론트/백엔드 코드와 Confluence를 탐색하여 원인을 분석합니다. "이슈 조사해줘", "버그 원인 찾아줘", "에러 추적해줘", "코드 찾아서 분석해줘" 등. Jira 이슈 키(PROJ-200 등)가 메시지에 포함되면 자동 트리거. 단순 코드 위치만 찾을 때는 find-in-code를 사용하세요.
user-invocable: true
argument-hint: [ISSUE_KEY, Jira URL, or keyword] (e.g., PROJ-200, "이미지 업로드 오류")
---

# 이슈 조사 스킬

Jira 이슈 키, Jira URL, 또는 키워드를 입력받아 코드베이스와 Confluence를 탐색하고 원인을 분석한다.
**Jira 댓글 작성은 하지 않는다** — 분석 결과만 출력. 댓글은 사용자가 `/jira-write`로 별도 요청.

## 자동 트리거 조건

아래 패턴이 사용자 메시지에 포함되면 이 스킬을 트리거:

1. **Jira URL**: `{site}.atlassian.net/browse/XXX-000` 형태의 URL
2. **이슈 키 + 탐색 요청**: `PROJ-200 관련 코드 찾아봐`, `PROJ-41 분석해줘`
3. **이슈/에러 + 코드 추적 요청**: `"이 이슈 확인해봐"`, `"에러 원인 찾아줘"`, `"front/backend 찾아봐"`
4. **슬래시 커맨드**: `/investigate PROJ-200`

**트리거하지 않는 경우:**
- 단순 코드 위치만 찾는 질문 → `find-in-code`
- 이미 원인을 알고 구현 방안을 묻는 질문 → `propose-fix`
- Jira 댓글/티켓 작성 요청 → `jira-write`

## 입력 형태

```
/investigate PROJ-200
/investigate https://{site}.atlassian.net/browse/PROJ-200
/investigate 이미지 업로드 오류
/investigate PROJ-200 --front-only
/investigate PROJ-200 --back-only
```

## 조사 프로세스

### STEP 1: 컨텍스트 수집

**Jira 이슈 키 또는 URL이 있으면:**
- MCP `getJiraIssue`로 이슈 상세 조회 (summary, description, comments)
- URL에서 이슈 키 추출 (예: `.../browse/PROJ-200` → `PROJ-200`)
- 이슈에서 키워드 추출 (기능명, 에러 메시지, 관련 페이지 등)

**키워드만 있으면:**
- 키워드를 그대로 탐색 기준으로 사용

### STEP 2: 코드 탐색 (넓게 → 좁혀가기)

프론트/백엔드를 **병렬로** 탐색한다 (Agent/Explore 활용).
코드 위치 탐색은 `find-in-code` 스킬의 출력 형식(절대경로:라인번호, 연결 흐름)을 따른다.

**프론트엔드 탐색:**
- 관련 페이지/컴포넌트 (pages, components)
- API 호출 함수 (api/)
- 유틸/훅 (hooks, utils)
- 에러 메시지, validation 로직

**백엔드 탐색:**
- Controller/Handler (엔드포인트 매핑)
- Service/Repository 레이어
- 설정 파일 (application.yml, ingress 등)
- 에러 응답 형식

**탐색 키워드 전략:**
- 기능명 (한글/영문 모두)
- URL 경로, API 엔드포인트
- 에러 메시지 텍스트
- 관련 테이블/엔티티명

### STEP 3: Confluence 검색

- Atlassian MCP `searchAtlassian`으로 관련 문서 검색
- 동일/유사 이슈 해결 이력 확인
- 인프라 설정 문서 (ingress, 배포 설정 등) 확인
- 관련 문서 발견 시 `getConfluencePage`로 상세 내용 확인

### STEP 4: 분석 및 정리

**흐름 추적:**
- 사용자 액션 → 프론트 → API → 백엔드 → 외부 서비스 순서로 전체 경로 추적
- 각 단계에서 실패 시 어떤 에러가 발생하는지 분석

**UX 메시지 점검 (validation/에러 메시지가 있을 때):**
- 경계값 표현: "최대 N 이하"(중복) → "N 이하" 또는 "최대 N". "미만" 기본 권장
- 서버 에러가 사용자에게 전달되는지 (catch에서 console.log만 하는 패턴 ❌)
- 프론트/백엔드/Ingress 기준 불일치 여부

## 출력 형식

분석 깊이는 이슈 복잡도에 따라 조절 — 단순 코드 위치 찾기면 간결하게, 버그 분석이면 상세하게.

```markdown
## 조사 결과: [이슈 제목 또는 키워드]

### 이슈 요약
(Jira에서 가져온 경우) 타입, 상태, 설명 요약

### 관련 코드

#### 프론트엔드
| 파일 | 라인 | 역할 | 검증 |
|------|------|------|------|
| `경로:라인` | 코드 | 설명 | ✓ |

#### 백엔드
| 파일 | 라인 | 역할 | 검증 |
|------|------|------|------|
| `경로:라인` | 코드 | 설명 | ✓ |

### 흐름 추적
[사용자] → [프론트:컴포넌트] → [API] → [백엔드] → [외부서비스]
                                         ↓ 실패 시
                                   [에러 메시지: "..."]

### 원인 분석
| 가능성 | 원인 | 근거 | 확인 방법 |
|--------|------|------|----------|
| ★★★ | 설명 | 코드 위치 | 명령어/방법 |
| ★★☆ | 설명 | 코드 위치 | 명령어/방법 |

### 관련 문서 (Confluence)
- [문서 제목](URL) — 요약

### UX 메시지 점검 (해당 시)
| 위치 | 현재 메시지 | 문제 | 개선안 |
|------|-----------|------|--------|

### 해결 방향
(코드 수정 방향만 제시, 상세 구현은 /propose-fix로)

### 다음 단계
- 상세 구현 → `/propose-fix`
- Jira 댓글 → `/jira-write`
- 코드 수정 후 리뷰 → `/my-review`
```

## 규칙

1. **병렬 탐색**: 프론트/백엔드/Confluence를 가능한 동시에 검색 (Agent 병렬 호출)
2. **절대경로:라인번호** 형식 필수
3. **Read로 라인 검증** 후 표에 기재 (✓/✗)
4. **원인은 가능성 순서로 정렬** (★★★ > ★★☆ > ★☆☆)
5. **댓글 작성 안 함** — 사용자가 `/jira-write`로 별도 요청
6. Jira MCP 도구: Atlassian MCP가 연결되어 있으면 `getJiraIssue`로 이슈 조회. 도구명은 환경에 따라 다를 수 있으므로 사용 가능한 MCP 도구를 탐색하여 사용
7. Confluence MCP 도구: Atlassian MCP의 `searchAtlassian` → 상세는 `getConfluencePage` 사용
