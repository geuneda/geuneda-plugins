# 스킬 유지보수 가이드

이 스킬 자체를 개선하거나 수정할 때 참고하는 문서. 새 LLM 세션에서 컨텍스트 없이 시작해도 이 문서만 보면 안전하게 변경/배포할 수 있다.

## 스킬 파일이 사는 두 곳

**중요**: 같은 스킬이 두 위치에 존재한다. 변경 시 반드시 양쪽을 동기화한다.

| 위치 | 역할 | 경로 |
|---|---|---|
| 로컬 작업본 | 본인 환경에서 즉시 실행되는 사본 | `~/.claude/skills/plan-to-devspec/` |
| 마켓플레이스 git 클론 | 팀 배포 소스. git push 대상 | `~/Desktop/geuneda-plugins/plugins/plan-to-devspec/skills/plan-to-devspec/` |

GitHub 저장소: https://github.com/geuneda/geuneda-plugins (계정: `geuneda`, HTTPS push 가능)

## 변경 흐름 (표준)

1. **로컬에서 수정** — `~/.claude/skills/plan-to-devspec/SKILL.md` 또는 `references/*.md` 편집
2. **로컬에서 테스트** — 아래 "테스트 절차" 참조
3. **마켓플레이스로 동기화** — 단일 명령:
   ```bash
   cp ~/.claude/skills/plan-to-devspec/SKILL.md \
      ~/Desktop/geuneda-plugins/plugins/plan-to-devspec/skills/plan-to-devspec/SKILL.md
   cp ~/.claude/skills/plan-to-devspec/references/*.md \
      ~/Desktop/geuneda-plugins/plugins/plan-to-devspec/skills/plan-to-devspec/references/
   ```
4. **커밋 + push** —
   ```bash
   cd ~/Desktop/geuneda-plugins
   git add plugins/
   git commit -m "fix(plan-to-devspec): {요약}"
   git push
   ```
5. **팀원 업데이트** — 팀원은 `/plugin marketplace update geuneda-plugins` 한 번 실행

역방향(마켓플레이스에서 먼저 수정한 경우)도 동일하게 cp 방향만 반대로.

## 테스트 절차

### 테스트 디렉토리

`~/Desktop/plan-to-devspec-test/`

이 디렉토리에 `.plan-to-devspec.json`이 미리 작성되어 있다. 노션 데이터 소스 URL과 컬럼/상태 매핑이 잡혀 있어 별도 입력 없이 바로 실행 가능.

### 노션 테스트 데이터

데이터 소스: `collection://3662dc3e-f514-80eb-afe5-000b7e302c1d`

미리 만들어둔 6개 페이지:

| 순서 | 제목 | 의도된 결과 |
|---|---|---|
| 1 | 일일 출석 보상 시스템 | 통과 (명세서 생성) |
| 2 | 랜덤 박스 가챠 | 검토 필요 (모호 표현, 데이터 누락) |
| 3 | 친구 초대 보상 | 검토 필요 (데이터 정의 누락, 예외 케이스 누락) |
| 4 | 이벤트 룰렛 | 통과 (명세 가능) |
| 5 | 퀘스트 시스템 | 처리 안 함 (기획 전) |
| 6 | PvP 매칭 | 처리 안 함 (개발 진행중) |

### 테스트 초기화

테스트 후 모든 항목을 `기획 완료` 상태로 되돌리고 비고를 비우려면 노션 페이지에서 수동 처리하거나, 다음 페이지 ID에 대해 `notion-update-page`를 호출하면 된다:

- 1번: `3662dc3e-f514-8168-a4ed-e27c00eae9b5`
- 2번: `3662dc3e-f514-8195-ba92-e7ea966a32e8`
- 3번: `3662dc3e-f514-81bb-87b0-dedcf656c306`
- 4번: `3662dc3e-f514-817a-aaab-ce3edef407a7`

명세서 파일 삭제: `rm -rf ~/Desktop/plan-to-devspec-test/docs/dev-specs/*`

### 회귀 검증 체크리스트

스킬 수정 후 최소 다음을 확인:

**기본 워크플로우**
- [ ] `notion-search` + `data_source_url` 호출이 페이지 목록을 반환하는지
- [ ] 각 페이지 `notion-fetch` 시 properties(`상태`, `작업 순서`)가 정확히 파싱되는지
- [ ] 상태 `기획 완료`인 항목만 처리 대상으로 필터링되는지
- [ ] 작업 순서 오름차순 정렬이 정확한지
- [ ] 1번(통과 케이스)에서 명세서가 `docs/dev-specs/`에 생성되는지
- [ ] 1번 처리 후 노션 상태가 `개발 진행중`으로 변경되는지
- [ ] 2번(검토 필요)에서 비고가 카테고리 포맷대로 작성되는지
- [ ] 2번에서 노션 상태가 `검토 필요`로 변경되는지
- [ ] 2번 검토 필요 발생 후 3번 이후로 진행하지 않고 중단되는지

**검토 분류 정확도** (잘못된 검토 요청 방지)
- [ ] 시트 관리 대상 데이터(확률표/가격/보상 풀)는 검토 필요로 분류하지 않는지
- [ ] 저장 위치(서버/로컬)는 `project_pattern.storage` 값으로 자동 결정하는지
- [ ] DB 스키마 디테일(테이블명/필드 타입)은 검토 필요로 분류하지 않는지
- [ ] 표준 UX(로딩 인디케이터/에러 토스트/멱등성/재시도)는 검토 필요로 분류하지 않는지
- [ ] 검토 비고 텍스트가 기획자 친화적 표현인지 (개발 용어 최소화, 의도 명확)
- [ ] 명세서에 `(추론: ...)` 표기로 추론 근거가 명시되는지

**답변 자동 반영 흐름** (검토 완료 → 기획 완료)
- [ ] `검토 완료` 상태 항목이 검토 대상보다 먼저 처리되는지
- [ ] 비고 텍스트가 `insert_content`로 기획서 본문 끝의 `## 검토 답변 ({날짜})` 섹션에 정확히 들어가는지
- [ ] 답변 반영 후 비고가 `> 답변 반영 완료 ({날짜})` 마커로 갱신되는지
- [ ] 답변 반영 후 상태가 `기획 완료`로 변경되는지
- [ ] 재검토 시 본문 + 답변 섹션을 함께 읽고 통과/재요청 판정하는지
- [ ] 재검토에서 검토 필요로 떨어질 때 이미 답변된 항목을 반복 요청하지 않는지

## 변경 시 주의사항

### 노션 도구 호출 형태 검증
- `notion-search`는 `data_source_url`을 받으면 해당 데이터 소스 내 페이지만 검색
- `notion-update-page`는 `command`, `properties`, `content_updates`, `position` 모두 필수
- `notion-fetch`는 `view://` URL을 지원하지 않음 (400 에러)
- `notion-query-data-sources` 도구는 현재 MCP에 없음

### 호환성
- 노션 데이터 소스 schema(`기획서`/`상태`/`작업 순서`/`비고`)가 바뀌면 `references/project_config.md`의 schema 매핑 안내도 갱신
- 상태 옵션 이름(특히 띄어쓰기)이 바뀌면 모든 곳에서 일치 확인

### 안전 규칙
- TC 데이터 소스(`collection://3662dc3e-f514-809e-...`), QA 데이터 소스(`collection://3662dc3e-f514-801c-...`)는 절대 읽기/쓰기 금지
- 기존 페이지 update만 허용, 새 페이지 생성 금지 (테스트 데이터 추가 같은 경우 명시적으로 사용자가 요청한 경우만)
- 토큰/시크릿은 어떤 파일에도 기록하지 않음

## 변경 이력 기록

큰 변경(워크플로우 추가/제거, 검토 카테고리 변경 등)은 커밋 메시지 본문에 다음 포맷으로 기록한다:

```
fix(plan-to-devspec): {짧은 요약}

변경 동기: {왜}
변경 내용:
- {무엇이 바뀌었는지}
주의사항: {호환성/마이그레이션 필요한 점}
테스트: {어떻게 검증했는지}
```

이렇게 적으면 `git log`로 추적 가능해 별도 CHANGELOG 파일이 필요 없다.

## 자주 묻는 변경 시나리오

### 시나리오 1: 검토 카테고리 추가
1. `references/review_checklist.md` 에 카테고리 H 추가 (기준 + 비고 작성 예시)
2. SKILL.md 본문은 보통 변경 불필요 (참조만 함)
3. 회귀 검증 시 새 카테고리를 트리거하는 테스트 데이터 추가

### 시나리오 2: 다른 노션 컬럼 이름을 쓰는 프로젝트 지원
1. `references/project_config.md`의 `schema` 매핑 활용법 명확히 (이미 있음)
2. SKILL.md의 노션 도구 호출 부분에서 하드코딩된 이름(`상태`, `작업 순서`) 대신 설정 매핑 사용
3. 테스트는 컬럼명을 바꾼 별도 데이터 소스로

### 시나리오 3: 명세서 템플릿 변경
1. `references/dev_spec_template.md` 수정
2. 1번 항목(`일일 출석 보상 시스템`) 처리 후 생성된 파일을 보고 의도대로 나오는지 확인

### 시나리오 4: 새 출력 형식 지원 (예: 노션 페이지 생성)
1. 신규 출력 형식을 SKILL.md Step 3-3에 분기로 추가
2. 사용자 설정(`.plan-to-devspec.json`)에 `output_mode: "markdown" | "notion-page"` 같은 옵션 추가
3. `references/project_config.md`에 옵션 설명 추가
4. 기본값은 기존 `markdown` 유지 (하위 호환)

## 권한/접근

- GitHub `geuneda` 계정으로 push 가능 (gh CLI HTTPS 토큰 인증)
- SSH 키는 `geunedaTS` 계정에 묶여 있어 origin을 SSH로 두면 push 실패
- 현재 `~/Desktop/geuneda-plugins`의 origin은 HTTPS여야 함. 만약 SSH로 바뀌어 push가 막히면:
  ```bash
  git -C ~/Desktop/geuneda-plugins remote set-url origin https://github.com/geuneda/geuneda-plugins.git
  ```
