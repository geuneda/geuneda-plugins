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
    "dev_in_progress": "개발 진행중",
    "dev_done": "개발 완료"
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

## 신규 프로젝트 설정 절차

설정 파일이 없을 때:

1. 사용자에게 데이터베이스 또는 데이터 소스 URL을 묻는다.
2. 받은 URL이 데이터베이스면 `notion-fetch`로 데이터 소스 목록을 조회해 기획용 데이터 소스를 식별한다 (사용자에게 확인).
3. 데이터 소스 스키마를 조회해 컬럼명(`기획서`, `상태`, `작업 순서`, `비고`)이 일치하는지 확인한다. 다르면 사용자에게 매핑을 묻는다.
4. 상태 옵션 5종(`기획 전`, `기획 완료`, `검토 필요`, `개발 진행중`, `개발 완료`)이 존재하는지 확인한다. 빠진 게 있으면 사용자에게 안내한다.
5. `output_dir` 기본값을 `docs/dev-specs`로 제안하고 사용자가 다르게 원하면 변경한다.
6. 위 내용을 모두 모아 `.plan-to-devspec.json`에 저장한다.

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
