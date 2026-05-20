# 스킬 유지보수 가이드

이 스킬 자체를 개선하거나 수정할 때 참고하는 문서. 새 LLM 세션에서 컨텍스트 없이 시작해도 이 문서만 보면 안전하게 변경/배포할 수 있다.

## 스킬 파일이 사는 곳

| 위치 | 역할 | 경로 |
|---|---|---|
| 마켓플레이스 git 클론 | 팀 배포 소스. git push 대상 | `~/.claude/plugins/marketplaces/geuneda-plugins/plugins/tc-dev-writer/` |

GitHub 저장소: https://github.com/geuneda/geuneda-plugins

기본적으로 마켓플레이스 클론에서 직접 수정하고 commit + push 하는 방식이다 (plan-to-devspec과 다르게 별도 로컬 작업본을 두지 않음). 팀원은 `/plugin marketplace update geuneda-plugins` 한 번 실행하면 받아간다.

## 변경 흐름 (표준)

1. **마켓플레이스 클론에서 직접 수정** —
   ```
   ~/.claude/plugins/marketplaces/geuneda-plugins/plugins/tc-dev-writer/skills/tc-dev-writer/SKILL.md
   ~/.claude/plugins/marketplaces/geuneda-plugins/plugins/tc-dev-writer/skills/tc-dev-writer/references/*.md
   ```
2. **로컬 테스트** — 아래 "테스트 절차" 참조
3. **커밋 + push** —
   ```bash
   cd ~/.claude/plugins/marketplaces/geuneda-plugins
   git add plugins/tc-dev-writer
   git commit -m "fix(tc-dev-writer): {요약}"
   git push
   ```
4. **팀원 업데이트** — 팀원은 `/plugin marketplace update geuneda-plugins` 실행

## 테스트 절차

### 테스트 환경 준비

테스트 디렉토리에서 `.tc-dev-writer.json`을 미리 만들어둔다:

```json
{
  "project_name": "RocketDan",
  "data_source_url": "collection://3662dc3e-f514-809e-aca8-000bbc3d90ba",
  "data_source_id": "3662dc3e-f514-809e-aca8-000bbc3d90ba",
  "database_url": "https://www.notion.so/teamsparta/TC-3662dc3ef514809794fdc48f174450a6",
  "schema": {
    "title_property": "번호",
    "date_property": "날짜",
    "content_property": "내용",
    "type_property": "선택",
    "priority_property": "우선순위",
    "status_property": "상태"
  },
  "type_values": { "bug": "버그", "improvement": "개선" },
  "priority_values": { "urgent": "당장", "high": "상", "medium": "중", "low": "하" },
  "status_values": { "todo": "진행 전", "in_progress": "진행 중", "done": "완료", "cant_handle": "처리 불가" },
  "default_status": "진행 전"
}
```

### 회귀 검증 체크리스트

스킬 수정 후 최소 다음을 확인:

**기본 워크플로우**
- [ ] 충분한 정보가 담긴 입력에서 역질문 없이 바로 등록 단계로 진행되는지
- [ ] 부족한 입력에서 역질문이 카테고리별로 묶여 한 번에 던져지는지
- [ ] 사용자 답변 후 TC가 4개 항목(항목/재현조건/절차/기대vs실제)을 모두 채우는지
- [ ] 등록 직전 사용자 확인 단계가 빠짐없이 나오는지
- [ ] `notion-create-pages` 호출 시 `parent.data_source_id`가 대시 포함 UUID인지 (collection:// 접두 없는지)
- [ ] 신규 페이지의 상태가 `진행 전`으로 설정되는지

**자동 판단 정확도**
- [ ] 명백한 버그 (크래시/오류/누락) → `버그` 분류
- [ ] 명백한 개선 (편의/일관성/추가) → `개선` 분류
- [ ] 우선순위 입력값이 있을 때 자동 판단으로 덮어쓰지 않는지
- [ ] 진행 불가 케이스가 `당장`으로 분류되는지
- [ ] 사소한 UI 케이스가 `하`로 분류되는지

**안전 규칙**
- [ ] TC 외 데이터소스(기획/QA 템플릿)를 호출하지 않는지
- [ ] 페이지 본문(content)에 코드가 안 들어가는지
- [ ] 사용자 명시적 요청 없이 중복 등록을 시도하지 않는지

### 테스트 후 정리

테스트로 만든 TC 페이지는 노션에서 직접 삭제하거나, 상태를 `처리 불가`로 두고 비고에 "테스트 데이터" 표시.

## 변경 시 주의사항

### 노션 도구 호출 형태 검증

- `notion-create-pages`는 `parent.data_source_id`에 **대시 포함 UUID**를 받는다. `collection://` 접두 붙이면 오류.
- Date 컬럼은 expanded format: `date:{property}:start`, `date:{property}:is_datetime`.
- Select 타입(`선택`, `우선순위`, `상태`)은 옵션 이름을 정확히 띄어쓰기까지 일치시켜야 한다.
- `notion-update-page`는 `command`, `properties`, `content_updates`, `position` 모두 필수.

### 호환성

- 노션 TC 데이터소스 schema(컬럼명/옵션명)가 바뀌면 `references/project_config.md`의 schema/values 매핑 안내도 갱신.
- 새 옵션이 추가되면(예: 우선순위에 `긴급` 추가) `references/classification_rules.md`의 자동 판단 트리 갱신.

### 안전 규칙

- TC 외 데이터소스 호출 금지:
  - 기획: `collection://3662dc3e-f514-80eb-afe5-000b7e302c1d`
  - QA 템플릿: `collection://3662dc3e-f514-801c-9d9b-000ba701f95b`
- 새 페이지 생성만 허용. 기존 페이지 수정은 사용자가 명시 요청한 경우만.
- 토큰/시크릿은 어떤 파일에도 기록하지 않음.

## 변경 이력 기록

큰 변경(워크플로우 추가/제거, 자동 판단 룰 변경 등)은 커밋 메시지 본문에 다음 포맷으로 기록한다:

```
fix(tc-dev-writer): {짧은 요약}

변경 동기: {왜}
변경 내용:
- {무엇이 바뀌었는지}
주의사항: {호환성/마이그레이션 필요한 점}
테스트: {어떻게 검증했는지}
```

이렇게 적으면 `git log`로 추적 가능해 별도 CHANGELOG 파일이 필요 없다.

## 자주 묻는 변경 시나리오

### 시나리오 1: 새 우선순위 옵션 추가

1. 노션에서 옵션 추가 (예: `긴급`)
2. `references/project_config.md`의 `priority_values` 매핑 갱신
3. `references/classification_rules.md`의 우선순위 의사결정 트리에 새 단계 추가
4. SKILL.md Step 5의 자동 판단 룰 설명도 동일하게 갱신

### 시나리오 2: 컬럼명이 다른 프로젝트 지원

1. `references/project_config.md`의 schema 매핑 활용 안내 명확히 (이미 있음)
2. SKILL.md의 노션 도구 호출 부분에서 하드코딩된 컬럼명(`내용`, `날짜` 등) 대신 설정 매핑 사용
3. 테스트는 컬럼명을 바꾼 별도 데이터소스로

### 시나리오 3: TC 본문 포맷 변경

1. `references/tc_writing_guide.md`의 출력 포맷 섹션 수정
2. 예시 TC 한 건을 직접 등록해 보고 의도대로 나오는지 확인

### 시나리오 4: 자동 판단 룰 보완

1. `references/classification_rules.md`의 해당 §1 또는 §2 갱신
2. 회귀 검증 시 새 룰을 트리거하는 케이스로 테스트

## 권한/접근

- GitHub `geuneda` 계정으로 push 가능 (gh CLI HTTPS 토큰 인증)
- 현재 origin이 SSH인지 HTTPS인지 확인 후 푸시 (`git -C ~/.claude/plugins/marketplaces/geuneda-plugins remote -v`)
- SSH로 push가 안 되면 HTTPS로 전환:
  ```bash
  git -C ~/.claude/plugins/marketplaces/geuneda-plugins remote set-url origin https://github.com/geuneda/geuneda-plugins.git
  ```
