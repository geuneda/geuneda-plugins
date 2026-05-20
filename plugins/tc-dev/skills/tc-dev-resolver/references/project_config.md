# 프로젝트별 설정

`tc-dev-resolver`는 `tc-dev-writer`와 **동일한 설정 파일**을 공유한다. 파일 위치·포맷·필드는 모두 같다. 한쪽 스킬에서 등록한 TC를 다른 쪽 스킬에서 처리하는 흐름이므로 따로 두면 어긋날 수 있어 의도적으로 단일 파일을 사용한다.

## 설정 파일 위치

스킬은 두 경로에서 설정을 찾는다:

1. **현재 작업 디렉토리**: `./.tc-dev-writer.json` (프로젝트 루트)
2. **유저 글로벌**: `~/.claude/tc-dev-writer.config.json`

프로젝트 루트 설정이 우선한다. 둘 다 없으면 `tc-dev-writer`의 `references/project_config.md`에 있는 **신규 프로젝트 설정 절차**(템플릿 복제 또는 기존 URL 입력 두 옵션)를 그대로 실행해 설정 파일을 만든다. 파일명은 `tc-dev-writer`로 그대로 두는 것이 권장 — 같은 데이터소스에 대해 두 스킬이 같은 진실을 본다.

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
    "processed_date_property": "처리 날짜",
    "content_property": "내용",
    "type_property": "선택",
    "priority_property": "우선순위",
    "status_property": "상태",
    "cause_property": "원인"
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
  "default_status": "진행 전",
  "naming_pattern": "TC-{YYYYMMDD}-{slug}"
}
```

### 처리 스킬이 추가로 참조하는 필드

| 필드 | 사용처 |
|------|--------|
| `schema.processed_date_property` | 처리 완료 시 날짜를 쓰는 컬럼명 (기본 `처리 날짜`) |
| `schema.cause_property` | 처리 완료/불가 사유를 쓰는 컬럼명 (기본 `원인`) |
| `status_values.done` | 처리 완료 상태 옵션명 (기본 `완료`) |
| `status_values.cant_handle` | 처리 불가 상태 옵션명 (기본 `처리 불가`) |

기존 설정 파일에 `processed_date_property`/`cause_property`가 없으면 기본값(`처리 날짜` / `원인`)을 사용한다. 컬럼명이 다른 프로젝트라면 사용자에게 매핑을 묻고 설정 파일을 갱신한다.

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

처리자가 쓰는 컬럼은 `처리 날짜`, `원인`, `상태` 셋뿐이다. 나머지(`내용`, `선택`, `우선순위`, `날짜`, `번호`)는 등록자(`tc-dev-writer`)의 영역이므로 절대 갱신하지 않는다.

## 페이지 속성 갱신 호출 예시

처리 완료 케이스:

```
notion-update-page
  page_id: "https://www.notion.so/teamsparta/TC-20260520-...-abc123"
  command: "update_properties"
  properties:
    date:처리 날짜:start: "2026-05-20"
    date:처리 날짜:is_datetime: 0
    원인: "Assets/Scripts/Boss/UltimateSkill.cs:142의 분기 순서가 잘못돼 게이지가 애니메이션보다 먼저 초기화되던 문제"
    상태: "완료"
```

처리 불가 케이스:

```
notion-update-page
  page_id: "https://www.notion.so/teamsparta/TC-20260520-...-abc123"
  command: "update_properties"
  properties:
    date:처리 날짜:start: "2026-05-20"
    date:처리 날짜:is_datetime: 0
    원인: "결제 모듈은 서버측 응답에 의존하므로 클라이언트 단독 수정 불가. 서버팀 결제 로그 확인 필요."
    상태: "처리 불가"
```

### 주의

- `page_id`는 페이지의 URL이나 UUID 그대로. `data_source_id`처럼 collection:// 접두는 붙이지 않는다.
- Select 타입(`상태`)의 옵션 이름은 정확히 일치 — 띄어쓰기 한 칸도 다르면 안 된다.
- `처리 날짜`는 새로 채우는 컬럼이므로 expanded format 모두(`:start`, `:is_datetime`)를 같이 보낸다.
- `command: "update_properties"`는 명시적으로 보낸다 (생략하면 본문 갱신으로 해석될 수 있음).

## 기준 프로젝트 (참고)

게임팀 - 로켓단 게임즈의 TC 페이지:
- 데이터베이스 URL: `https://www.notion.so/teamsparta/TC-3662dc3ef514809794fdc48f174450a6`
- 데이터베이스에는 3개 데이터소스가 있다 (템플릿 / TC / 기획)
- **TC 데이터소스**: `collection://3662dc3e-f514-809e-aca8-000bbc3d90ba` (이 스킬의 처리 대상)
- 다른 데이터소스(기획, QA 템플릿)는 절대 읽거나 수정하지 않는다

## 보안/안전 규칙

- 설정 파일에는 토큰이나 시크릿을 저장하지 않는다 (URL과 매핑 정보만).
- 데이터소스 URL은 변경 가능. 설정 파일을 수정하면 즉시 반영된다.
- 같은 프로젝트에서 여러 TC 데이터소스를 다뤄야 하면 사용자에게 매번 어떤 설정을 쓸지 확인한다.
- `tc-dev-writer`와 컬럼 매핑이 어긋나면 등록은 되는데 처리에서 못 찾는 사고가 생긴다 — schema 갱신 시 양쪽을 같이 본다.
