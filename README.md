GitHub: https://github.com/soomin1996/ledger-api · Render: https://ledger-api-vnht.onrender.com (API 문서: https://ledger-api-vnht.onrender.com/docs)

# 가계부 API — FastAPI + Supabase(PostgreSQL)

클라우드컴퓨팅실습 W4 과제. FastAPI + SQLAlchemy로 계좌·카테고리·거래를 Supabase(PostgreSQL)에 저장·조회·집계하고, Render에 배포한다.

## 엔드포인트

| 메서드 | 경로 | 설명 |
|---|---|---|
| POST | `/accounts` | 계좌 생성 |
| GET | `/accounts` | 계좌 목록 |
| GET | `/accounts/{account_id}` | 계좌 단건 |
| POST | `/transactions` | 거래 생성 (없는 계좌면 404) |
| GET | `/accounts/{account_id}/detail` | 계좌 + 거래 목록 중첩 응답 |
| GET | `/stats/by-category` | 카테고리별 지출 합계 (GROUP BY) |

## 실습 기록

### ① 결과 확인

- 로컬(`127.0.0.1:8000`)에서 계좌 1개(월급통장), 카테고리 2개(식비·교통), 거래 2건(점심 -12,000 / 지하철 -1,500)을 넣었고 Supabase Table Editor에서 확인했다.
- `GET /stats/by-category` 결과: `[{"category":"교통","total":-1500,"count":1},{"category":"식비","total":-12000,"count":1}]`
- Render 배포 후 `GET /accounts`가 로컬에서 만든 계좌(월급통장)를 그대로 돌려줬고, 배포 주소에서 `POST /accounts`로 만든 「배포테스트」 계좌가 Supabase `accounts` 테이블에 바로 나타났다 → 로컬 앱과 Render 앱이 같은 Supabase DB를 본다.
- (캡처 추가 예정: Supabase Table Editor, Render `/docs`의 `GET /accounts`)

### ② 핵심 개념 되새김

- **계좌·거래를 두 테이블로 나눈 이유 (1:N)**: 계좌 하나에 거래가 여러 건 붙는다. 거래마다 계좌 정보를 복사하지 않고 `account_id`(외래키)로 가리키면 계좌 정보를 한 곳에서만 관리하면 된다.
- **모델 클래스 ↔ 테이블**: `class Account(Base)` 하나가 `accounts` 테이블 하나이고, `Mapped[...]` 속성 하나가 컬럼 하나다. `create_all()`이 이 클래스를 보고 `CREATE TABLE`을 대신 실행한다.
- **접속 문자열을 .env로 분리하는 이유**: 비밀번호가 들어 있어서 GitHub에 올리면 안 되고, 로컬(.env)과 Render(환경변수)에서 코드는 그대로 둔 채 값만 바꿔 끼울 수 있다.

### ③ 자유 로그

- (본인 기록으로 채우기: 배운 것 / 막힌 곳과 해결 / 아직 안 풀린 것)
- 예: Supabase 대시보드에서 연결 문자열을 어디서 찾는지 막혔다 → `Connect` → Method를 `Session pooler`로 바꿔서 해결.
- AI 활용: Claude Code에게 워크북 코드로 파일 생성과 연결 테스트를 맡겼고, `/accounts/1/detail`, `/stats/by-category` 응답을 워크북의 예상 결과와 대조하고 Supabase Table Editor에서 행을 직접 확인해 검증했다.
