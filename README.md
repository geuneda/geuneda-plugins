# geuneda-plugins

이종근 개인 Claude Code 플러그인 마켓플레이스. 게임팀에서 공유해서 사용한다.

## 등록된 플러그인

| 이름 | 설명 |
|------|------|
| `plan-to-devspec` | 노션 기획 데이터베이스에서 기획 완료 항목을 읽어 개발 명세서(마크다운)로 변환 |

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
```

설치 후 `/plan-to-devspec` 또는 자연어로 "기획서 변환해줘" 식으로 호출하면 된다.

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
    └── plan-to-devspec/
        ├── .claude-plugin/
        │   └── plugin.json           # 플러그인 메타
        └── skills/
            └── plan-to-devspec/
                ├── SKILL.md          # 스킬 본문 (YAML frontmatter + 마크다운)
                └── references/       # 스킬에서 참조하는 보조 문서
                    ├── dev_spec_template.md
                    ├── project_config.md
                    └── review_checklist.md
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
