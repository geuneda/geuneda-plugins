# geuneda-plugins

이종근 개인 Claude Code 플러그인 마켓플레이스. 게임팀에서 공유해서 사용한다.

## 등록된 플러그인

| 이름 | 설명 | 사용 데이터 소스 |
|------|------|------------------|
| `plan-to-devspec` | 노션 기획 데이터베이스에서 기획 완료 항목을 읽어 개발 명세서(마크다운)로 변환. 모호한 항목은 노션 상태를 `검토 필요`로 바꾸고 비고에 질문을 남긴다. | 기획 |
| `tc-dev` | 노션 TC(테스트케이스) 통합 플러그인. `tc-dev-writer`로 사용자가 발견한 현상을 개발자 작업 가능한 형태로 구체화해 등록하고, `tc-dev-resolver`로 TC 번호를 받아 분석·코드 수정·처리 날짜·원인·상태를 자동 갱신한다. | TC |
| `qa-testcase-notion` | 개발 완료된 기능의 코드 변경(커밋/PR diff)을 QA 친화 표현으로 변환해 Notion TC 템플릿 DB에 행을 자동 추가. 환경 클러스터링 3단계 방법론으로 케이스 순서를 최적화한다. | QA (템플릿) |
| `data-validator` | Google Sheets 게임 데이터 테이블을 스키마/크로스참조/밸런스 이상치 3종으로 검증한 뒤 Slack 알림. 매일 오전 10시 KST 자동 실행 및 `/validate-data` 수동 트리거 지원. | - |

## 노션 데이터베이스 템플릿

`plan-to-devspec`, `tc-dev`, `qa-testcase-notion` 세 플러그인은 **동일한 노션 데이터베이스 한 곳을 공유**해서 동작한다. 이 데이터베이스에는 3개 데이터 소스(View)가 들어 있다.

### 템플릿 위치

```
https://www.notion.so/teamsparta/3662dc3ef51480e4aa0afd6780ebcea5
```

(게임팀 - 로켓단 게임즈 / [직무] 개발 / [개발] TC 템플릿 하위)

### 데이터 소스 매핑

| View 이름 | 용도 | 사용 플러그인 |
|-----------|------|---------------|
| `기획` | 기획서 작성 및 검토/개발 진행 상태 추적 | `plan-to-devspec` |
| `TC` | 버그/개선 TC 발견·등록·처리 | `tc-dev` (writer/resolver) |
| `QA` | 개발 완료 기능의 QA 전달용 테스트케이스 표 | `qa-testcase-notion` |

각 데이터 소스의 컬럼(스키마)은 정해져 있으므로 임의로 바꾸면 플러그인이 동작하지 않는다. 컬럼 매핑은 각 플러그인의 `references/project_config.md`(또는 `SKILL.md`)를 참조.

### 프로젝트 첫 실행 동작 — DB 자동 설정 흐름

플러그인 첫 실행 시 설정 파일(`./.{plugin}.json` 또는 `~/.claude/{plugin}.config.json`)이 없으면 다음과 같이 묻는다.

```
[데이터베이스가 없습니다]
어떻게 진행할까요?

1) 에이전트가 템플릿을 복제해 새 DB를 만든다
   - 복제될 부모 페이지 URL을 입력 받음
   - 위 템플릿을 notion-duplicate-page로 복제
   - 새 데이터베이스에서 해당 플러그인용 데이터 소스 URL 자동 추출
2) 이미 만들어둔 DB의 URL을 직접 입력한다
   - 데이터베이스 또는 데이터 소스 URL 입력
```

선택 후 플러그인이 데이터 소스를 식별·검증하고 설정 파일을 자동 생성한다. 한 번 설정해두면 이후엔 묻지 않는다. 자세한 절차는 각 플러그인의 `references/project_config.md` 참조.

**참고**: 같은 프로젝트에서 세 플러그인을 모두 쓰려면 **하나의 데이터베이스만 복제**하면 된다. 그 데이터베이스의 세 데이터 소스를 세 플러그인이 각각 사용한다. 각 플러그인이 본인 설정 파일에 자기 데이터 소스 URL만 저장한다.

## 팀원 사용 방법

Claude Code에서 한 번만 등록하면 이후 자동 업데이트된다.

### 1. 마켓플레이스 추가

Claude Code 실행 후:

```
/plugin marketplace add geuneda/geuneda-plugins
```

또는 SSH로:

```
/plugin marketplace add git@github.com:geuneda/geuneda-plugins.git
```

### 2. 플러그인 설치

```
/plugin install plan-to-devspec@geuneda-plugins
/plugin install tc-dev@geuneda-plugins
/plugin install qa-testcase-notion@geuneda-plugins
/plugin install data-validator@geuneda-plugins
```

설치 후 자연어 또는 슬래시 커맨드로 호출한다. 노션을 쓰는 세 플러그인은 첫 실행 시 위 [DB 자동 설정 흐름](#프로젝트-첫-실행-동작--db-자동-설정-흐름)을 따른다.

### 3. 업데이트

```
/plugin marketplace update geuneda-plugins
```

리포지토리에 새 커밋이 푸시되면 위 명령으로 받아올 수 있다.

## 기여 방법

### 0. 권한 확인

이 저장소는 `geuneda` 개인 소유 public 저장소다. 기여 방식은 두 가지:

- **Collaborator로 직접 push**: `geuneda`에게 collaborator 추가 요청 → 브랜치 생성 후 PR
- **Fork → PR**: 본인 계정으로 fork → 수정 → upstream으로 PR

작은 수정이라도 main 브랜치에 직접 push하지 말고 반드시 PR로 진행한다.

### 1. 클론 및 브랜치 생성

```bash
git clone git@github.com:geuneda/geuneda-plugins.git
cd geuneda-plugins

# 작업 유형별 브랜치 이름 규칙
git checkout -b add-skill/{스킬명}        # 새 스킬 추가
git checkout -b add-plugin/{플러그인명}   # 새 플러그인 추가
git checkout -b fix/{요약}                # 버그 수정
git checkout -b update/{요약}             # 기존 스킬/플러그인 수정
```

### 2. 디렉토리 구조

```
geuneda-plugins/
├── .claude-plugin/
│   └── marketplace.json              # 마켓플레이스 메타 + 플러그인 목록
└── plugins/
    ├── plan-to-devspec/
    ├── tc-dev/
    ├── qa-testcase-notion/
    └── data-validator/
        └── (각 플러그인) .claude-plugin/plugin.json + skills/{스킬명}/
```

### 3. 기존 플러그인에 새 스킬 추가

가장 일반적인 케이스. 예: `plan-to-devspec` 플러그인에 `another-skill`을 추가.

```bash
mkdir -p plugins/plan-to-devspec/skills/another-skill/references
```

`plugins/plan-to-devspec/skills/another-skill/SKILL.md` 작성:

```markdown
---
name: another-skill
description: 이 스킬을 언제 사용하는지 한두 문장으로. 트리거 키워드도 포함. 예: "X를 처리할 때 사용. 'X 처리', 'do X' 등의 요청 시 활성화한다."
---

# 스킬 제목

스킬 본문. 워크플로우, 규칙, 예시 등을 마크다운으로 작성.
```

- `name`은 디렉토리명과 정확히 일치 (소문자, 하이픈 가능)
- `description`은 Claude가 언제 이 스킬을 호출할지 판단하는 근거이므로 구체적으로 작성
- 본문이 길어지면 `references/` 폴더에 분리하고 SKILL.md에서 참조 명시

### 4. 새 플러그인 추가

플러그인은 관련된 스킬을 묶는 단위. 도메인이 다르면 새 플러그인으로 분리한다.

```bash
mkdir -p plugins/{플러그인명}/.claude-plugin
mkdir -p plugins/{플러그인명}/skills/{스킬명}/references
```

`plugins/{플러그인명}/.claude-plugin/plugin.json`:

```json
{
  "name": "플러그인명",
  "description": "이 플러그인이 다루는 영역 설명",
  "author": { "name": "본인 GitHub 이름" },
  "repository": "https://github.com/geuneda/geuneda-plugins",
  "keywords": ["키워드1", "키워드2"]
}
```

그 다음 `.claude-plugin/marketplace.json`의 `plugins` 배열에 엔트리 추가:

```json
{
  "name": "플러그인명",
  "source": "./plugins/플러그인명",
  "description": "마켓플레이스에 표시될 짧은 설명",
  "keywords": ["키워드1", "키워드2"],
  "category": "productivity"
}
```

배열 마지막 엔트리 뒤에 쉼표를 붙이지 않도록 주의 (JSON 파싱 오류).

### 5. JSON 유효성 검증

플러그인 메타와 마켓플레이스 파일은 JSON 문법이 깨지면 전체 마켓플레이스가 로드되지 않는다. 커밋 전 반드시 확인:

```bash
python3 -m json.tool .claude-plugin/marketplace.json > /dev/null && echo OK
python3 -m json.tool plugins/{플러그인명}/.claude-plugin/plugin.json > /dev/null && echo OK
```

### 6. 로컬 테스트

PR 올리기 전에 본인 Claude Code에서 동작을 확인:

```bash
# 작업 디렉토리에서 Claude Code 실행
claude --plugin-dir ./plugins/{플러그인명}
```

이후 `/플러그인명:스킬명` 또는 자연어 트리거로 호출해 의도대로 동작하는지 점검.

### 7. 커밋 메시지

Conventional Commits 형식, 한국어 본문 권장.

```
feat({플러그인명}): {스킬명} 스킬 추가
fix({플러그인명}): {요약}
docs: README 업데이트
refactor({플러그인명}): {요약}
```

예시:
```
feat(plan-to-devspec): 검토 필요 항목 자동 슬랙 알림 추가
fix(plan-to-devspec): 작업 순서 동일값 처리 시 정렬 오류 수정
```

### 8. Push 및 PR

```bash
git push -u origin {브랜치명}
gh pr create --title "feat({플러그인명}): {요약}" --body "변경 내용 요약..."
```

또는 GitHub 웹에서 PR 생성. 머지되면 모든 팀원이 `/plugin marketplace update geuneda-plugins`로 받아간다.

### 9. 머지 후

- 사용자는 명시적으로 `update` 명령을 실행해야 새 버전을 받는다 (자동 업데이트 아님)
- 큰 변경(스킬 동작 방식 변경, breaking change) 시 슬랙으로 공지
- 의도치 않은 동작이 발견되면 빠르게 hotfix PR 생성

## 라이선스

Internal use only.
