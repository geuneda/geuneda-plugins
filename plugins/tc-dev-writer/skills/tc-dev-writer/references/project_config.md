# 프로젝트별 설정

각 프로젝트마다 다른 노션 TC 데이터소스를 사용한다. 설정 파일을 통해 프로젝트 식별 후 해당 데이터소스에 TC를 등록한다.

## 설정 파일 위치

스킬은 두 경로에서 설정을 찾는다:

1. **현재 작업 디렉토리**: `./.tc-dev-writer.json` (프로젝트 루트)
2. **유저 글로벌**: `~/.claude/tc-dev-writer.config.json`

프로젝트 루트 설정이 우선한다. 둘 다 없으면 사용자에게 데이터소스 URL을 물어 입력받은 뒤 프로젝트 루트(또는 사용자가 원하는 곳)에 저장한다.

## 설정 파일 포맷

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
  "type_values": {
    "bug": "버그",
    "improvement": "개선"
  },
  "priority_values": {
    "urgent": "당장",
    "high": "상",
    "medium": "중",
    "low": "하"
  },
  "status_values": {
    "todo": "진행 전",
    "in_progress": "진행 중",
    "done": "완료",
    "cant_handle": "처리 불가"
  },
  "default_status": "진행 전"
}
```

### 필드 설명

| 필드 | 필수 | 설명 |
|------|------|------|
| `project_name` | O | 프로젝트 식별용 이름 |
| `data_source_url` | O | TC 데이터소스 URL (collection:// 형식). `notion-search`의 `data_source_url` 파라미터에 그대로 넣어 기존 번호 조회 |
| `data_source_id` | O | TC 데이터소스 ID (대시 포함, `notion-create-pages` parent용) |
| `database_url` | △ | 데이터베이스 페이지 URL (참고용) |
| `schema.*` | O | 노션 컬럼명 매핑 (프로젝트마다 다를 수 있음) |
| `type_values.*` | O | `선택` 컬럼의 옵션 이름 매핑 (버그/개선) |
| `priority_values.*` | O | `우선순위` 컬럼의 옵션 이름 매핑 (당장/상/중/하) |
| `status_values.*` | O | `상태` 컬럼의 옵션 이름 매핑 |
| `default_status` | O | 신규 등록 시 기본 상태 (보통 `진행 전`) |
| ~~`naming_pattern`~~ | ✗ | **폐기됨.** 번호는 더 이상 명명 규칙 기반이 아니라 (기존 정수 최대값 + 1)로 자동 부여. 이 키가 설정에 남아 있어도 무시된다. |

## 신규 프로젝트 설정 절차

설정 파일이 없을 때:

1. 사용자에게 노션 TC 데이터베이스 또는 데이터소스 URL을 묻는다.
2. 받은 URL이 데이터베이스면 `notion-fetch`로 데이터소스 목록을 조회해 TC용 데이터소스를 식별한다 (사용자에게 확인).
3. 데이터소스 schema를 조회해 컬럼명이 일치하는지 확인한다 (`번호`, `날짜`, `내용`, `선택`, `우선순위`, `상태`). 다르면 사용자에게 매핑을 묻는다.
4. 선택 옵션(`버그`, `개선`), 우선순위 옵션(`당장`, `상`, `중`, `하`), 상태 옵션(`진행 전`, `진행 중`, `완료`, `처리 불가`)이 존재하는지 확인한다. 빠진 게 있으면 사용자에게 안내한다.
5. 위 내용을 모두 모아 `.tc-dev-writer.json`에 저장한다.

## TC 데이터소스 schema (기준 프로젝트)

게임팀 - 로켓단 게임즈의 TC 데이터소스 schema:

```sql
CREATE TABLE "collection://3662dc3e-f514-809e-aca8-000bbc3d90ba" (
    url TEXT UNIQUE,
    createdTime TEXT,
    "date:날짜:start" TEXT,
    "date:날짜:end" TEXT,
    "date:날짜:is_datetime" INTEGER,
    "date:처리 날짜:start" TEXT,
    "date:처리 날짜:end" TEXT,
    "date:처리 날짜:is_datetime" INTEGER,
    "상태" TEXT,         -- ["진행 전", "진행 중", "완료", "처리 불가"]
    "선택" TEXT,         -- ["개선", "버그"]
    "원인" TEXT,
    "내용" TEXT,
    "우선순위" TEXT,     -- ["상", "중", "하", "당장"]
    "번호" TEXT          -- title
)
```

`번호`가 title 컬럼이므로 페이지의 title 자리에 들어간다. **운영 규칙상 정수 문자열(`"1"`, `"2"`, ...)을 순차 부여**한다 — 신규 등록 시 `notion-search`로 기존 번호 중 정수 최대값을 찾아 +1. `원인`과 `처리 날짜`는 담당자가 TC를 처리하며 채우는 컬럼이라 **신규 등록 시 비워둔다**.

### 다음 번호 조회 호출 예시

```
notion-search
  query: "TC"           # semantic search — 값 자체는 부수적, data_source_url 필터가 핵심
  data_source_url: "collection://3662dc3e-f514-809e-aca8-000bbc3d90ba"
  page_size: 25
  filters: {}
  max_highlight_length: 0
```

응답의 `results[*].title`을 정수로 파싱해 최대값 + 1. `has_more`이면 `start_cursor`로 페이지네이션.

## 페이지 생성 호출 예시

```
notion-create-pages
  parent:
    type: "data_source_id"
    data_source_id: "3662dc3e-f514-809e-aca8-000bbc3d90ba"
  pages:
    - properties:
        번호: "6"   # = 기존 정수 최대값(5) + 1
        date:날짜:start: "2026-05-20"
        date:날짜:is_datetime: 0
        선택: "버그"
        우선순위: "상"
        내용: "[항목] 보스전 - 캐릭터 궁극기\n[재현 조건] iOS, 챕터 5 보스전..."
        상태: "진행 전"
      content: ""
```

### 주의: data_source_id 형식

`parent.data_source_id`에는 **대시 포함 UUID**만 넣는다. `collection://` 접두를 붙이면 오류가 발생한다.

- ✅ `3662dc3e-f514-809e-aca8-000bbc3d90ba`
- ❌ `collection://3662dc3e-f514-809e-aca8-000bbc3d90ba`

## 기준 프로젝트 (참고)

게임팀 - 로켓단 게임즈의 TC 페이지:
- 데이터베이스 URL: `https://www.notion.so/teamsparta/TC-3662dc3ef514809794fdc48f174450a6`
- 데이터베이스에는 3개 데이터소스가 있다 (템플릿 / TC / 기획)
- **TC 데이터소스**: `collection://3662dc3e-f514-809e-aca8-000bbc3d90ba` (이 스킬의 등록 대상)
- 다른 데이터소스(기획, QA 템플릿)는 절대 수정하지 않는다

## 보안/안전 규칙

- 설정 파일에는 토큰이나 시크릿을 저장하지 않는다 (URL과 매핑 정보만).
- 데이터소스 URL은 변경 가능. 설정 파일을 수정하면 즉시 반영된다.
- 같은 프로젝트에서 여러 TC 데이터소스를 다뤄야 하면 사용자에게 매번 어떤 설정을 쓸지 확인한다.
