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

### 새 스킬 추가

```bash
git clone git@github.com:geuneda/geuneda-plugins.git
cd geuneda-plugins

# 새 플러그인을 추가하는 경우
mkdir -p plugins/{플러그인명}/.claude-plugin
mkdir -p plugins/{플러그인명}/skills/{스킬명}/references

# 기존 plugin-to-devspec에 스킬을 추가하는 경우
mkdir -p plugins/plan-to-devspec/skills/{새스킬명}
```

브랜치 생성 → 작업 → PR. 머지되면 모든 사용자가 다음 `/plugin marketplace update`에서 받는다.

### 디렉토리 구조

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

## 라이선스

Internal use only.
