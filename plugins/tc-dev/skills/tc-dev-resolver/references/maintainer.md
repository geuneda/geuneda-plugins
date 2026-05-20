# 스킬 유지보수 가이드

이 스킬 자체를 개선하거나 수정할 때 참고하는 문서. 새 LLM 세션에서 컨텍스트 없이 시작해도 이 문서만 보면 안전하게 변경/배포할 수 있다.

## 스킬 파일이 사는 곳

| 위치 | 역할 | 경로 (예) |
|---|---|---|
| 사용자 로컬 작업본 | 직접 수정 및 git push 대상 | `/Users/<id>/geuneda-plugins/plugins/tc-dev/skills/tc-dev-resolver/` |
| (참고) 마켓플레이스 캐시 | 팀원이 `update` 후 받아가는 캐시 | `~/.claude/plugins/marketplaces/geuneda-plugins/` |

GitHub 원격: upstream `https://github.com/geuneda/geuneda-plugins`, 개인 fork는 origin.

`tc-dev-writer`와 동일 플러그인(`tc-dev`) 아래에 있다. 사용자 로컬 작업본에서 수정하고 origin에 push, PR로 upstream 머지. 팀원은 `/plugin marketplace update geuneda-plugins`로 받는다.

## tc-dev-writer와의 관계

- **같은 플러그인의 자매 스킬**: 두 스킬은 모두 `plugins/tc-dev/skills/` 아래에 있고 같은 `plugin.json`을 공유한다.
- 두 스킬은 **같은 노션 데이터소스**를 다룬다. 컬럼 매핑 / 옵션값이 한쪽에서만 바뀌면 어긋난다.
- 동일한 설정 파일(`.tc-dev-writer.json` 또는 `~/.claude/tc-dev-writer.config.json`)을 공유한다 — 의도된 설계다. 파일을 두 개로 쪼개지 않는다.
- `tc-dev-writer`는 페이지 생성, `tc-dev-resolver`는 페이지 속성 갱신. 권한 분리가 분명하므로 양쪽이 같은 페이지를 동시에 건드릴 일은 거의 없다.
- schema/옵션 변경 시 항상 두 스킬의 `references/project_config.md`를 같이 갱신한다 — 짝지어 다닐 것.

## 변경 흐름 (표준)

1. **로컬 작업본에서 직접 수정** —
   ```
   <작업본>/plugins/tc-dev/skills/tc-dev-resolver/SKILL.md
   <작업본>/plugins/tc-dev/skills/tc-dev-resolver/references/*.md
   ```
2. **로컬 테스트** — 아래 "테스트 절차" 참조
3. **커밋 + push** —
   ```bash
   git add plugins/tc-dev/skills/tc-dev-resolver
   git commit -m "fix(tc-dev-resolver): {요약}"
   git push origin <branch>
   ```
   두 스킬을 동시에 손봤다면 `git add plugins/tc-dev`로 한 번에 묶어도 무방하다.
4. **팀원 업데이트** — 팀원은 `/plugin marketplace update geuneda-plugins` 실행 (변경이 upstream에 머지된 후)

## 테스트 절차

### 테스트 환경 준비

`tc-dev-writer`의 테스트와 동일하게 `.tc-dev-writer.json`을 준비. 추가로 처리 컬럼 매핑이 있어도 되고, 없으면 기본값(`처리 날짜`, `원인`)을 사용한다.

```json
{
  "project_name": "RocketDan",
  "data_source_url": "collection://3662dc3e-f514-809e-aca8-000bbc3d90ba",
  "data_source_id": "3662dc3e-f514-809e-aca8-000bbc3d90ba",
  "database_url": "https://www.notion.so/teamsparta/TC-3662dc3ef514809794fdc48f174450a6",
  "schema": {
    "title_property": "번호",
    "date_property": "날짜",
    "processed_date_property": "처리 날짜",
    "content_property": "내용",
    "type_property": "선택",
    "priority_property": "우선순위",
    "status_property": "상태",
    "cause_property": "원인"
  },
  "type_values": { "bug": "버그", "improvement": "개선" },
  "priority_values": { "urgent": "당장", "high": "상", "medium": "중", "low": "하" },
  "status_values": { "todo": "진행 전", "in_progress": "진행 중", "done": "완료", "cant_handle": "처리 불가" },
  "default_status": "진행 전"
}
```

테스트는 실제 노션 TC 데이터에 영향을 주므로, 미리 "테스트용 TC" 페이지 한 건을 만들어 두고 그것만 가지고 굴린다. 처리 후 노션에서 다시 `진행 전` 상태/빈 처리 컬럼으로 되돌려 다음 테스트에 재사용.

### 회귀 검증 체크리스트

스킬 수정 후 최소 다음을 확인:

**기본 워크플로우**
- [ ] 번호 정확 매칭으로 1건이 잡힐 때 추가 질문 없이 Step 4로 진행하는지
- [ ] 번호가 부분 일치만 잡힐 때 폴백 검색이 작동하는지
- [ ] 2건 이상 매칭 시 사용자에게 후보를 보여주고 확인을 받는지
- [ ] 0건 매칭 시 적절한 안내로 종료하는지
- [ ] 이미 `완료`/`처리 불가` 상태인 TC를 자동으로 재처리하지 않는지
- [ ] `내용`이 표준 포맷에서 벗어났을 때 임의 보정하지 않고 사용자에게 확인하는지
- [ ] 코드 수정 전 사용자에게 변경 의도를 한 줄로 공지하는지
- [ ] 노션 갱신 직전 사용자 확인 단계가 빠짐없이 나오는지
- [ ] `notion-update-page` 호출 시 `처리 날짜`, `원인`, `상태`만 갱신하고 다른 컬럼은 건드리지 않는지

**원인 식별 정확도**
- [ ] 분기 조건 오류 / 호출 순서 / 상수 잘못 케이스에서 위치를 정확히 지목하는지
- [ ] 가설이 검증되지 않을 때 `처리 불가`로 분기하는지 (추측 패치 만들지 않는지)
- [ ] 외부 의존(서버/SDK/아트) 케이스가 `처리 불가`로 분류되는지
- [ ] 명세 결정 필요 케이스가 `처리 불가`로 분류되고 PM/기획 확인 안내가 나오는지

**원인 컬럼 품질**
- [ ] 원인 한 줄에 파일:라인이 포함되는지 (코드 수정 시)
- [ ] 처리 불가 사유에 "필요 조치"가 포함되는지
- [ ] 진단 형태(why)이지 결과 보고(what)가 아닌지

**안전 규칙**
- [ ] TC 외 데이터소스(기획/QA 템플릿)를 호출하지 않는지
- [ ] 페이지 생성/삭제를 시도하지 않는지
- [ ] 사용자 명시 요청 없이 같은 페이지를 중복 갱신하지 않는지
- [ ] 노션 갱신 실패 시 코드 변경을 임의 롤백하지 않는지

### 테스트 후 정리

테스트로 갱신한 노션 페이지는 수동으로 `진행 전` 상태로 되돌리고 `원인`, `처리 날짜`를 비운다. 이를 위해 노션 페이지를 별도로 한 건 마련해 두는 것이 효율적이다.

## 변경 시 주의사항

### 노션 도구 호출 형태 검증

- `notion-search`의 `data_source_url`에는 `collection://` 접두를 포함한다.
- `notion-update-page`는 `command`, `properties`(필요 시 `content_updates`, `position`도)를 명시한다. 속성 갱신은 `command: "update_properties"`.
- Date 컬럼은 expanded format: `date:{property}:start`, `date:{property}:is_datetime`.
- Select 타입(`상태`)은 옵션 이름을 정확히 띄어쓰기까지 일치시켜야 한다 — `진행 전`, `진행 중`, `완료`, `처리 불가`.

### 호환성

- 노션 TC 데이터소스 schema(컬럼명/옵션명)가 바뀌면 `tc-dev-resolver`와 `tc-dev-writer` 양쪽의 `references/project_config.md`를 같이 갱신.
- 새 상태 옵션이 추가되면(예: `이관` 등) `references/project_config.md`와 SKILL.md의 갱신 로직 분기 갱신.
- `tc-dev-writer`가 만드는 `내용` 포맷이 바뀌면 `references/content_parsing.md`도 같이 갱신해야 처리 시 파싱이 어긋나지 않는다.

### 안전 규칙

- TC 외 데이터소스 호출 금지:
  - 기획: `collection://3662dc3e-f514-80eb-afe5-000b7e302c1d`
  - QA 템플릿: `collection://3662dc3e-f514-801c-9d9b-000ba701f95b`
- 페이지 생성/삭제 권한 없음. 속성 갱신만.
- 토큰/시크릿은 어떤 파일에도 기록하지 않음.

## 변경 이력 기록

큰 변경은 커밋 메시지 본문에 다음 포맷으로:

```
fix(tc-dev-resolver): {짧은 요약}

변경 동기: {왜}
변경 내용:
- {무엇이 바뀌었는지}
주의사항: {호환성/마이그레이션 필요한 점}
테스트: {어떻게 검증했는지}
```

## 자주 묻는 변경 시나리오

### 시나리오 1: 새 상태 옵션 추가 (예: '이관')

1. 노션에서 옵션 추가
2. `references/project_config.md`의 `status_values` 매핑 갱신 (tc-dev-writer도 같이)
3. SKILL.md Step 9의 상태 옵션 후보에 추가
4. Step 6/Step 9의 분기 트리에 새 상태로 가는 경로가 필요한지 검토 (보통 처리 불가 외 분기는 추가하지 않는 방향이 안전)

### 시나리오 2: `tc-dev-writer`가 만드는 `내용` 포맷 변경

1. `tc-dev-writer`의 `references/tc_writing_guide.md` 변경 확인
2. 이쪽 `references/content_parsing.md`의 포맷 예시와 섹션 매핑 갱신
3. 회귀 검증: 새 포맷으로 등록된 TC를 처리해 봤을 때 작업 단위 추출이 정상인지

### 시나리오 3: 처리 불가 분기 카테고리 추가

1. SKILL.md Step 6의 처리 불가 분기 조건에 추가
2. `references/cause_writeback.md`의 처리 불가 사유 한 줄 작성 패턴에 새 카테고리 추가
3. `references/resolution_workflow.md`의 처리 불가 분기 섹션 갱신

### 시나리오 4: 컬럼명이 다른 프로젝트 지원

1. `references/project_config.md`의 schema 매핑 명확히 (이미 있음 — `processed_date_property`, `cause_property` 등)
2. SKILL.md의 노션 도구 호출 부분에서 하드코딩된 컬럼명(`처리 날짜`, `원인`) 대신 설정 매핑 사용
3. 테스트는 컬럼명을 바꾼 별도 데이터소스로

## 권한/접근

- GitHub `geuneda` 계정으로 push 가능 (gh CLI HTTPS 토큰 인증)
- 현재 origin이 SSH인지 HTTPS인지 확인 후 푸시 (`git -C ~/.claude/plugins/marketplaces/geuneda-plugins remote -v`)
- SSH로 push가 안 되면 HTTPS로 전환:
  ```bash
  git -C ~/.claude/plugins/marketplaces/geuneda-plugins remote set-url origin https://github.com/geuneda/geuneda-plugins.git
  ```
