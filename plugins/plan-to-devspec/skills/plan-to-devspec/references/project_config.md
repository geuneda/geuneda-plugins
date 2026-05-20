# 프로젝트별 데이터베이스 설정

각 프로젝트마다 다른 노션 데이터베이스(기획 데이터 소스)를 사용한다. 설정 파일을 통해 프로젝트 식별 후 해당 데이터 소스 URL로 작업한다.

## 설정 파일 위치

스킬은 두 경로에서 설정을 찾는다:

1. **현재 작업 디렉토리**: `./.plan-to-devspec.json` (프로젝트 루트)
2. **유저 글로벌**: `~/.claude/plan-to-devspec.config.json`

프로젝트 루트 설정이 우선한다. 둘 다 없으면 사용자에게 데이터 소스 URL을 물어 입력받은 뒤 프로젝트 루트에 저장한다.

## 설정 파일 포맷

```json
{
  "project_name": "RocketDan",
  "data_source_url": "collection://3662dc3e-f514-80eb-afe5-000b7e302c1d",
  "database_url": "https://www.notion.so/teamsparta/TC-3662dc3ef514809794fdc48f174450a6",
  "output_dir": "docs/dev-specs",
  "schema": {
    "title_property": "기획서",
    "status_property": "상태",
    "order_property": "작업 순서",
    "note_property": "비고"
  },
  "status_values": {
    "before_planning": "기획 전",
    "planning_done": "기획 완료",
    "needs_review": "검토 필요",
    "review_answered": "검토 완료",
    "dev_in_progress": "개발 진행중",
    "dev_done": "개발 완료"
  },
  "project_pattern": {
    "storage": "server",
    "data_managed_in_sheet": true,
    "sheet_url": ""
  }
}
```

### 필드 설명

| 필드 | 필수 | 설명 |
|------|------|------|
| `project_name` | O | 프로젝트 식별용 이름 |
| `data_source_url` | O | 기획 데이터 소스 URL (collection:// 형식) |
| `database_url` | △ | 데이터베이스 페이지 URL (참고용) |
| `output_dir` | O | 개발 명세서 저장 경로 (프로젝트 루트 기준 상대 경로) |
| `schema.*` | O | 노션 컬럼명 매핑 (프로젝트마다 다를 수 있음) |
| `status_values.*` | O | 노션 상태 옵션 이름 매핑 |
| `project_pattern.storage` | O | `"server"` 또는 `"local"`. 영구 데이터의 저장 위치 패턴 |
| `project_pattern.data_managed_in_sheet` | O | 기획 데이터(밸런싱/콘텐츠)를 구글 시트로 관리하는지 |
| `project_pattern.sheet_url` | △ | 시트 관리 시 시트 URL (참고용, 없어도 됨) |

### project_pattern 사용 방식

이 값들은 검토 로직과 명세서 작성 모두에 영향을 미친다.

**검토 시**:
- `data_managed_in_sheet=true` 이면 확률표/가격/보상 풀 같은 시트 관리 데이터를 검토 필요로 처리하지 않는다 (review_checklist.md §1 참조).
- `storage` 값에 따라 저장 위치 결정을 자동 추론하므로 검토 필요로 처리하지 않는다 (review_checklist.md §2 참조).

**명세서 작성 시** (dev_spec_template.md 데이터 모델 섹션):
- 시트 관리 데이터: "구글 시트로 관리됨" 표시
- 저장 위치: `storage` 값에 따라 자동 표기 (`서버 DB (영구)` 또는 `로컬 저장 (PlayerPrefs/SQLite/JSON 중 프로젝트 표준)`)
- 보안 민감 데이터(결제/재화/보상): `storage`와 무관하게 서버 권위
- 초기화돼도 되는 임시 데이터: 캐시

## 신규 프로젝트 설정 절차

설정 파일이 없을 때 사용자에게 다음과 같이 안내한다.

```
[데이터베이스가 없습니다]
plan-to-devspec은 노션 기획 데이터 소스를 사용합니다. 어떻게 진행할까요?

1) 에이전트가 템플릿을 복제해 새 DB를 만든다
2) 이미 만들어둔 DB의 URL을 직접 알려준다
```

`AskUserQuestion`으로 두 옵션을 제시한다 ("Other"로 사용자가 직접 답할 수도 있음). 입력에 따라 아래 두 흐름 중 하나를 따른다.

### 옵션 1: 템플릿 복제 (에이전트가 생성)

1. 사용자에게 **복제될 부모 페이지 URL**을 묻는다 ("이 데이터베이스를 어느 페이지 아래에 만들까요?").
2. 다음 템플릿 데이터베이스를 `notion-duplicate-page`로 복제한다.
   - 템플릿 URL: `https://www.notion.so/teamsparta/3662dc3ef51480e4aa0afd6780ebcea5`
   - `parent`에 사용자가 알려준 페이지 ID 전달
3. 복제 결과로 받은 새 데이터베이스 URL을 `notion-fetch`로 조회해 데이터 소스 목록을 얻는다. 데이터 소스 중 컬럼 `기획서/상태/작업 순서/비고`가 일치하는 것이 **기획용 데이터 소스**.
4. 사용자에게 새 데이터베이스 URL과 식별된 데이터 소스를 보여주고 진행을 알린다 (확인 질문은 하지 않는다 — 단순 통보).
5. 옵션 2의 3~6 단계로 이동해 스키마 검증과 설정 저장을 수행한다.

### 옵션 2: 기존 URL 입력 (사용자가 생성)

1. 사용자에게 **데이터베이스 또는 데이터 소스 URL**을 묻는다.
2. 받은 URL이 데이터베이스면 `notion-fetch`로 데이터 소스 목록을 조회해 기획용 데이터 소스를 식별한다 (사용자에게 확인).
3. 데이터 소스 스키마를 조회해 컬럼명(`기획서`, `상태`, `작업 순서`, `비고`)이 일치하는지 확인한다. 다르면 사용자에게 매핑을 묻는다.
4. 상태 옵션 6종(`기획 전`, `기획 완료`, `검토 필요`, `검토 완료`, `개발 진행중`, `개발 완료`)이 존재하는지 확인한다. 빠진 게 있으면 사용자에게 노션 UI에서 추가하라고 안내한다 (`status` 타입은 MCP로 옵션 추가 불가).
5. `output_dir` 기본값을 `docs/dev-specs`로 제안하고 사용자가 다르게 원하면 변경한다.
6. 위 내용을 모두 모아 `.plan-to-devspec.json`에 저장한다.

## 페이지 목록 조회 (데이터 소스 행 수집)

노션 MCP에는 데이터 소스의 모든 행을 직접 가져오는 도구가 없다. 다음 우회 방법을 사용한다.

### 방법 1: notion-search (권장)

`data_source_url`을 지정해 해당 데이터 소스 내에서 검색.

```
notion-search
  query: "시스템 보상"              # semantic search이므로 데이터 소스 도메인 일반 키워드
  data_source_url: collection://...
  page_size: 25                     # 최대값
  max_highlight_length: 0           # 응답 크기 절약
  filters: {}
```

반환되는 `results` 배열의 각 항목은 `{id, title, url, type, timestamp}` 만 포함한다. `상태`, `작업 순서` 같은 properties는 포함되지 않으므로, 각 페이지를 `notion-fetch`로 한 번 더 조회해 properties를 확인해야 한다.

### query 선택 팁

- 첫 시도는 데이터 소스 제목/주제와 관련된 일반 단어 ("기획", "시스템", "보상" 등)
- 너무 특이한 단어를 쓰면 일부 페이지가 누락될 수 있음
- 결과 수가 적으면 다른 query로 재검색해 합집합으로 사용

### 방법 2: 사용자 직접 입력 (페이지 25개 초과 / search 결과가 신뢰 안 갈 때)

```
사용자에게 페이지 URL 목록을 줄바꿈으로 받아 그 목록을 처리 대상으로 사용
```

### 방법 3: notion-fetch + view URL (미지원)

`view://...` URL을 fetch에 넘기면 400 에러가 발생하므로 사용 불가.

## 페이지 properties 확인

`notion-search` 결과의 페이지 ID를 `notion-fetch`로 조회하면 properties가 함께 반환된다:

```
<properties>
{"기획서":"일일 출석 보상 시스템","상태":"기획 완료","작업 순서":1,"비고":""}
</properties>
```

이 값으로 `상태=="기획 완료"` 필터링 및 `작업 순서` 정렬을 수행한다.

## 기준 프로젝트 (참고)

예시로 사용되는 게임팀 - 로켓단 게임즈의 TC 페이지:
- 데이터베이스 URL: `https://www.notion.so/teamsparta/TC-3662dc3ef514809794fdc48f174450a6`
- 데이터베이스에는 3개 데이터 소스(템플릿/TC/기획)가 있다
- 기획 데이터 소스: `collection://3662dc3e-f514-80eb-afe5-000b7e302c1d`
- 다른 데이터 소스(TC, QA)는 절대 수정하지 않는다

## 보안/안전 규칙

- 설정 파일에는 토큰이나 시크릿을 저장하지 않는다 (URL과 매핑 정보만).
- 데이터베이스 URL은 변경 가능. 설정 파일을 수정하면 즉시 반영된다.
- 같은 프로젝트에서 여러 데이터베이스를 다뤄야 하면 사용자에게 매번 어떤 설정을 쓸지 확인한다.
