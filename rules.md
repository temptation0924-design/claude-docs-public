# rules.md — 하위원칙 + 자주 실수 패턴

> **원본 위치**: `~/.claude/rules.md` | **원본 리포**: `temptation0924-design/claude-system-docs` (private) | **공개 미러**: `claude-docs-public`
> **마지막 동기화**: 2026-08-08 v2.6 — 세션 tracker 워크트리 스코프 전환 + SESSION_RESUME 접기(전역오염·노이즈 근본수정, A16·B2 경로 참조 갱신) (구 v2.4: A5 모델 팔레트 Opus 5 갱신)

---

## A. 하위원칙

운영 지침의 세부 실행 규칙 (모드/스킬/루틴으로 이관된 원칙들의 하위 규정).

<!-- id:R-A1 -->
### A1. 파일명 규칙

**적용 대상**: 문서/설정 파일 중 **작업 산출물** (기획서, 보고서, 인수인계, 설계 문서 등)

**형식**: `[파일명]_YYYYMMDD_v1.확장자` — 날짜 + 버전 모두 포함
- 예시:
  - `세션인수인계_20260412_1차_v1.md`
  - `프로젝트계획_아이리스_20260412_v1.md`
  - `보고서_임대점검_20260412_v2.md`

**다운로드 파일명 패턴**: 기존 프로젝트의 파일명 패턴을 먼저 확인하고 따를 것 (`프로젝트명_YYYYMMDD_설명_v1.확장자`)

**인수인계 파일 특별 규칙**
- 형식: `세션인수인계_YYYYMMDD_N차_v1.md`
- 저장 위치: `~/.claude/handoffs/` (2026-04-11 v4.2.2부터 전용 디렉토리)

<!-- id:R-A10 -->
**🚫 대상에서 제외 (버전/날짜 불필요)**
1. **코드 파일**: `.py`, `.sh`, `.js`, `.ts`, `.html`, `.css`, `.tsx`, `.go` 등 실행 파일
2. **시스템/고정 이름 파일** (이름이 고정이어야 작동):
   - `CLAUDE.md`, `rules.md`, `session.md`, `skill-guide.md`, `env-info.md`, `checklist.md`, `agent.md`
   - `settings.json`, `keybindings.json`, `package.json`, `.env*`
   - `README.md`, `SKILL.md`, `MEMORY.md`, `CHANGELOG.md`
3. **자동 생성/스키마 고정 파일**: 메모리 파일(`~/.claude/projects/.../memory/*.md`), 로그 파일

<!-- id:R-A2 -->
### A2. Notion 저장 규칙

<!-- id:R-A12 -->
**정해진 루틴 (바로 저장 — 묻지 말 것)**
- 세션 종료 시 작업기록 DB 저장
- 에러 발생 시 에러로그 DB 저장 — **L1~L3는 의무, L4는 선택** (아래 정의 참조)
- 규칙 위반 시 규칙위반 DB 기록 (반복횟수 +1)
- → 이미 라우팅 맵이 정해진 DB는 확인 없이 바로 기록

<!-- id:R-A11 -->
**에러 4단계 정의** (2026-04-19 v1.8 명문화 — 누락 방지)
- **L1 Exception/Crash**: tool 호출 실패, script 비정상 종료, hook block — **즉시 worklog ERROR append + 세션 종료 시 에러로그 DB 등록 의무**
- **L2 진단 오류**: 표면 에러를 잘못된 원인으로 결론 비약 (예: 404 → "환경 문제" 단정) — **인지 즉시 worklog + 등록 의무**
- **L3 시스템 결함**: 핵심 파일/스크립트 버그 (regex silent fail, fallback 누락 등) — **수정 commit 직후 worklog + 등록 의무**
- **L4 외부 의존성 실패**: API 5xx, MCP 미응답 등 — 단순 재시도로 복구 시 **선택**, 30분+ 영향 시 **L1로 승격**

→ 노션기록관(2)는 `$WORKLOG_FILE`(`code/session_paths.sh`로 해석 — 워크트리별 스코프)의 `ERROR:` 라인 수를 소스로 사용 (1건 이상 시 강제 dispatch).

**정해지지 않은 저장 (2단계 확인 필수)**
- (1) "저장할까요?" → 대표님 OK
- (2) "어디에 저장할까요?" → 대표님 지정
- **명시적 허락 없이 바로 저장 금지** (B6 재발 방지)

**DB 생성 전 중복 확인**: `notion-search`로 유사 DB 존재 여부 반드시 확인

<!-- id:R-A13 -->
**Notion MCP 알려진 버그 2종 회피** (2026-04-18 v1.7 매뉴얼화)
- Bug 1: `replace_content` 동일 8글자 URL prefix 중복 파싱 → `update_content` 우회
- Bug 2: `update_properties` relation single-value 거부 → 전체 null → 재입력
- **사전 차단 패턴 + 즉시 우회 절차**: [`docs/rules/notion-mcp-bugs.md`](docs/rules/notion-mcp-bugs.md) 참조
- 근거: `feedback_notion_mcp_parser_bug_v1.md`, `feedback_notion_relation_validation_bug_v1.md`

<!-- id:R-A3 -->
### A3. 스킬 적용 규칙

- **스킬 확인 순서**
  - **1차 매칭**: `skill-guide.md` 키워드 검색 → 명확한 매칭이면 즉시 invoke
  - **매칭 애매/없음**: → 아래 3단 종합 체크 진입

<!-- id:R-A14 -->
- **추천 우선순위 (3단 종합 체크)**: 아래 3가지를 모두 확인한 후 최적안 추천
  1. **기존 설치 스킬**: `skill-guide.md`에서 해당 작업에 쓸 수 있는 스킬 검색
  2. **GitHub 유사 스킬**: 기존 설치된 것 외에 GitHub에서 더 나은 유사 스킬이 있는지 탐색
  3. **업그레이드 요소 체크**: 기존 스킬이 구버전이거나 기능 부족이면 업그레이드 가능 여부 판단
  - → 세 가지 종합 결과로 **"기존 활용 / 업그레이드 권장 / 신규 설치 권장"** 중 택일 추천
  - **TODO**: 향후 `skill-manager` 스킬에 이 3단 체크 로직을 내장할 예정

- **설치 전 기존 패턴 확인**: 이전에 설치한 스킬(file-organizer, gstack 등)의 설치 패턴을 먼저 참고
- **설치 경로 확인**: CLAUDE.md와 skill-guide.md에 명시된 경로를 반드시 확인 후 설치

- **진입 시 매칭 (MODE 1 워크플로우 9번)**: 승인된 계획 기반으로 → **1%라도 매칭 가능성 있으면 3단 종합 체크 진입** → 결과에 따라 `기존 활용 / 업그레이드 / 신규 설치` 중 택일

- **완료 후 스킬화 (자동)**: **3단 종합 체크 후에도 적합 스킬 없음** → MODE 2 워크플로우 6번에서 "스킬로 만들까요?" 자동 질문 → 승인 시 `skill-manager`로 자동 생성 + `skill-guide.md` 자동 등록 (로컬 + Notion)

- **단순 운영 업무 (MODE 4)**: **1차 매칭만** 수행 (`skill-guide.md` 키워드 검색) — 3단 종합 체크는 MODE 1/2 본격 업무에만 적용 (단순 업무 효율 우선)

<!-- id:R-A4 -->
### A4. 세션 루틴 규칙

<!-- id:R-A15 -->
- **시작 루틴 (반드시 순서대로)**:
  1. 자주 실수 패턴 TOP 5 상기 — **Notion DB 동적 조회** (`⚠️ 규칙 위반 기록`, `해결여부=false` + `반복횟수 DESC` + `limit 5`) → 한 줄 다짐 출력
  2. **고정 인사 문구 출력**: "어떤 업무를 진행하세요? ☺️ 기획-실행-검증-운영모드 대기중입니다!"
  3. 대표님 답변 → **모드 라우팅 (MODE 1~4 판별)** + 도구 추천 + 스킬 매칭

<!-- id:R-A16 -->
- **종료 루틴 (반드시 전부 실행, 순서대로)**:
  1. 작업기록 DB 저장
  2. 에러로그 기록 (에러 있을 때)
  3. 세션 인수인계 `.md` 파일 생성 → `~/.claude/handoffs/세션인수인계_YYYYMMDD_N차_v1.md`
     - ⚠️ `session-end-check.sh` 훅이 자동 차단 (**B2 재발 방지 — 38회 반복 TOP 1 위반**)
     - ⚠️ **비정상 종료 대비**: Stop 훅 미발동(강제종료·크래시)으로 handoff 누락 시, 다음 SessionStart에서 직전 세션 handoff 존재 여부 확인 → 없으면 즉시 생성
  4. **다음세션인계 컬럼 기록** (Notion 작업기록 DB)
  5. **메모리 상태 반영** (MEMORY.md + 개별 메모리 파일) — 2026-04-11 v4.2.2부터 필수
  6. **세션 소요시간 계산** (`$SESSION_START_FILE` epoch 활용 — `eval "$(bash ~/.claude/code/session_paths.sh --export)"`로 해석. 워크트리별 스코프, 구 전역 경로 하드코딩 금지)
  7. **TOP 5 자체 점검** → 어긴 항목 있으면 Notion DB의 해당 `위반코드` row에 `반복횟수 +1` (신규 패턴이면 Select 옵션 추가 후 신규 row 생성)
     - ⏰ **맨 마지막에 점검**: 1~6 진행 중 드러난 위반까지 반영하기 위함
  8. Slack 알림 (Claude Code Agent → #general-mode `C0AEM5EJ0ES` private_channel) — 상세 작업일지 포맷은 [`docs/rules/slack-worklog.md`](docs/rules/slack-worklog.md) 참조

<!-- id:R-A5 -->
### A5. 모델 추천 규칙 (2026-07-07 개정 — 구 「도구 추천 규칙」 · 2026-08-08 팔레트 Opus 5 반영)

- **기본값**: 현재 세션 모델 유지 — 추천은 보조 신호
- **모델 팔레트 (4단)**
  - **Haiku 4.5** — 단순 조회·정형 반복·대량 처리 (파일 정리, 로그 파싱, 단건 DB 기록)
  - **Sonnet 5** — 일반 코딩·문서 작업·정기 운영 점검
  - **Opus 5** — 복잡한 코딩·리뷰·디버깅 (`/fast` 모드 지원)
  - **Fable 5** — 기획·아키텍처·고위험 변경·다단계 오케스트레이션 (최상위)
  - 📌 **버전 주기 (2026-08-08)**: 팔레트 4칸은 **등급 이름만** 담는다 — 세대 표기는 각 칸의 현행 모델 1개로 유지. Opus 칸은 **Opus 5**가 현행이며, 구세대 **Opus 4.8**은 선택 가능한 폴백(`/fast`는 5·4.8 양쪽 지원)이지 별도 등급이 아니다. 압축본(CLAUDE.md MODE 1 10번 · B4 가드 훅 · `agents/tool-advisor.md`)에는 등급 이름 4개만 싣고, 폴백·세대 정보는 이 항목에만 둔다

- **모드별 추천 시점**
  - **MODE 1 기획 (복잡 업무)**: 워크플로우 10번 (승인된 계획 기반) — 실행에 쓸 모델 추천
  - **MODE 2 실행 / MODE 3 검증**: MODE 1에서 결정된 모델 계승 (재추천 불필요)
  - **MODE 4 운영 (단순 업무)**: `session.md` 세션 시작 3번 — task 단위 즉시 추천

<!-- id:R-A17 -->
- **추천 형식**: `"이 작업은 **[모델]** 권장 (이유: ~)"`
  - ⚡ **완화 규칙 (2026-07-07)**: 현재 모델과 **다른** 모델이 적합할 때만 1줄 명시, 같으면 침묵 허용. 구 "자명해도 스킵 금지" 원칙은 B4 누적 50회 위반의 구조적 원인으로 판정되어 폐지

- **에이전트 dispatch 등급은 별개**: agent.md 모델 정책(팀원별 Haiku/Sonnet/Opus 등급)을 그대로 따름
- **도구(Code/Claude.ai/Cowork) 구분**: CLAUDE.md 도구 계층 표 참조 — 정기 추천 루틴에서는 제외 (2026-07-07)
- **대표님 선택 존중**: "OK" 또는 "아니, [다른 모델]로 해줘"

<!-- id:R-A6 -->
### A6. 배포/설치 경로 규칙

**배포 경로**
- **Railway (백엔드)**: `git push origin main` → auto-deploy
  - 대상: haemilsia-bot, API 서버, 봇 등
- **Netlify (프론트엔드)**: `git push origin main` → auto-deploy
  - 대상: 랜딩페이지, 정적 사이트

**설치/저장 경로**
- **스킬**: `~/.claude/skills/`
- **인수인계**: `~/.claude/handoffs/`
- **훅**: `~/.claude/hooks/`
- **규칙**: `~/.claude/rules/`
- **시스템 문서**: `~/.claude/` (루트)

<!-- id:R-A18 -->
**배포 전 체크리스트**
- **90% 룰**: preflight-check 종합 점수 90% 이상 ([`docs/rules/preflight-check.md`](docs/rules/preflight-check.md) 참조)
  - 공식: `100% - (CRITICAL × 15%) - (WARNING × 3%)`
  - 90% 미만 FAIL → 자동 수정 → 재검증 반복 (PASS까지)
- **훅 통과 확인**: B1/B2/B5/B8 자동 차단 훅 (`~/.claude/hooks/`)
- **배포 스킬 활용**
  - `/ship` — PR 생성 + 코드리뷰 + 푸시 (gstack 스킬)
  - `/land-and-deploy` — 머지 + 배포 + 헬스체크 (gstack 스킬)
  - `haemilsia-bot-deploy` — Railway 전용 봇 배포 가이드 (로컬 스킬)
  - `landing-page-deploy` — Netlify + Railway 랜딩페이지 배포 (로컬 스킬)

> **TODO** (별도 세션): 위 배포 스킬 4개 이름이 직관적이지 않아 재명명 예정. 변경 완료 후 A6 재수정.

<!-- id:R-A7 -->
### A7. Git → GitHub 동기화 규칙 (2026-04-12 v1.7 — Notion 개별 동기화 폐기)

**원칙**
- **Git (`~/.claude/`) = 유일한 원본** (Source of Truth)
- **GitHub raw URL 통합본** = Claude.ai 열람본
- **Notion 개별 백업 7페이지** = 2026-04-12 폐기 (비효율 — 16분+12만 토큰 소모)
- Notion은 **DB 기록 전용** (작업기록, 에러로그, 규칙 위반 기록)

**동기화 대상: 1개**

- `INTEGRATED.md` — 7개 시스템 문서 자동 concat (`build-integrated_v1.sh`)
- URL: `https://raw.githubusercontent.com/temptation0924-design/claude-docs-public/main/INTEGRATED.md`

**수정 흐름**
1. Git 파일 수정 (`~/.claude/*.md`)
2. `build-integrated_v1.sh --push` → GitHub `INTEGRATED.md` 재빌드 (~10초)

<!-- id:R-A19 -->
**B8 체크리스트 (자동화됨 v2 — 2026-04-19)**
- ✅ `debounce_sync.sh`가 수정 30초 후 자동 빌드+push
- ✅ 시크릿 스캔 게이트 (토큰 패턴 발견 시 push 차단)
- ✅ Stop 훅 fallback (debounce 못 돈 케이스 동기 실행)
- ✅ `override_flag: --force-B8` 제거 (뒷문 차단)
- ⚡ 긴급 우회: `SKIP_B8_AUTOSYNC=1` 환경변수 (로그 필수 기록)
- ⚠️ 실패 시만 수동: `bash ~/.claude/code/build-integrated_v1.sh --push`

<!-- id:R-A8 -->
### A8. 에이전트 디스패치 원칙 (2026-04-12 C+ 시스템 신설)

- **병렬이 가능하면 무조건 병렬** — 순차는 데이터 의존 시에만
- **실패 자동 승급**: Haiku→Sonnet→Opus. 대표님 개입은 Opus 실패 후에만
- **매니저는 조합만, 실행은 팀원** — 매니저가 직접 하는 건 대화/병합/라우팅뿐
- **수동 모드 진입 시** 매니저가 추천 1~3개 필수 제시 (이유 + 모델 등급 + 예상 소요)
- **Notion 읽기 실패** → 에스컬레이션 안 함. 1회 타임아웃 → 즉시 폴백 (캐시 참조)
- **에스컬레이션 로그** → 에러로그 DB 자동 기록
- **상세 규칙**: `docs/rules/agent-dispatch.md` 참조
- **에이전트 레지스트리**: `agent.md` 참조
- **에이전트 프로필**: `~/.claude/agents/*.md` (C+ 팀 19명 — GSD 전용 `gsd-*` 에이전트는 GSD 설치기가 별도 관리)

<!-- id:R-A9 -->
### A9. MEMORY.md 용량 한도 (2026-07-07 바이트 기준 개정 — 구 「줄 수 한도」)

<!-- id:R-A20 -->
- **한도**: MEMORY.md(핫) **총 8,000바이트 이하** (`wc -c` 기준)
- **개정 근거**: 구 80줄 한도는 실제 토큰 부하를 못 잡음 — 2026-07-07 점검에서 40줄인데 9.3KB(줄당 500자+ 장문 11줄) 사례. 바이트가 자동로드 비용의 실측 지표
- **줄당 권장**: 600바이트(한글 약 200자) 이하 — 초과 시 상세는 개별 카드/handoff 링크로 압축
- **🆕 핫/콜드 2단 구조 (2026-06-26 v2.5)**: 영속지식(feedback/project/reference)은 콜드 `MEMORY-knowledge.md`로 분리 — 자동로드 안 됨, `bash ~/.claude/code/recall.sh <키워드>`로 검색. 새 영속 카드의 인덱스 줄은 **콜드에 추가**(핫엔 금지). 핫엔 최근완료·할일·반복위반·Session end만.
- **초과 시 절차 (순서대로)**:
  1. 🟢 최근 완료에서 7일 지난 항목 롤링 제거
  2. 남은 장문 줄 압축 (핵심 1줄 + `[[카드]]` 링크로 상세 이관)
  3. 완료 project 항목 `~/.claude/projects/-Users-ihyeon-u--claude/memory/archive/`로 mv (구 조항의 `-Users-ihyeon-u/memory` 경로는 오기 — 2026-07-07 정정)
- **(no desc) 줄 금지**: 메모리 파일 추가 시 frontmatter description 필수 + MEMORY.md 인덱스도 동기 채우기
- **세션 종료 시 자동 점검**: 핸드오프작성관이 `wc -c` 측정 → 8,000바이트 초과 시 **그 자리에서 압축** (다음 세션 이월 금지)

<!-- id:R-A21 -->
### A21. 추측 금지 + 묻기 (Think Before Coding) — Karpathy 1번 / R-A21 (2026-05-08 v2.0 신설)

- **추측 금지**: 모르면 멈추고 질문. 혼란을 숨기지 말 것
- **트레이드오프 명시**: "A vs B 둘 다 가능, 선택 근거는 ~" 한 줄로 박제
- **가정 명시**: "전세는 월세 흐름 있다고 가정" 같은 식으로 구두 박제
- **다중 해석 발견 시**: 양쪽 다 보여주고 대표님 선택 받기 (혼자 한쪽 진행 X)
- **케이스**: 5/7 ACTIVE_LEASE_TYPES — "전세 포함?" 질문 없이 월세/반전세/단기임대 3종 가정 → 17건 누락. "전세 슬롯 정책은?" 1줄 질문이면 사고 막힘

<!-- id:R-A22 -->
### A22. 단순함 우선 (Simplicity First) — Karpathy 2번 / R-A22 (2026-05-08 v2.0 신설)

- **요구된 것만 작성**: 묻지 않은 기능, 추상화, "유연성" 금지
- **200줄을 50줄로 줄일 수 있으면 다시 써라** — 일회성 코드는 추상화 X
- **YAGNI 강제**: "나중에 쓸 수도 있어서" = 삭제. 현재 필요만 구현
- **케이스**: rent_slot Block Kit 작성 시 새 빌더 만들지 말고 `_build_building_blocks` 재사용 (이미 박제된 패턴 — feedback_block_kit_build_pattern)

<!-- id:R-A23 -->
### A23. 외과수술적 변경 (Surgical Changes) — Karpathy 3번 / R-A23 (2026-05-08 v2.0 신설)

- **건드릴 것만 건드린다**: 인접 코드 "개선" 금지, 기존 스타일 매치
- **불필요한 dead code 발견 시**: 언급만 하고 삭제 X (별도 사이클로)
- **임포트 정리**: 내 변경으로 unused가 된 것만 제거 (기존 unused 손대지 말 것)
- **케이스**: ACTIVE_LEASE_TYPES fix 시 인근 lease 분류 로직까지 손대고 싶어도 참고, fix 1건 + commit 1건으로 끝낸다. 큰 PR = 리뷰 지옥 + 머지 지연

<!-- id:R-A24 -->
### A24. 목표 주도 실행 (Goal-Driven Execution) — Karpathy 4번 / R-A24 (2026-05-08 v2.0 신설)

- **착수 전 검증 가능한 종료 기준 정의**: "검증" → "X grep 결과 ≥ N건" 같은 명령으로 변환
- **PASS까지 루프**: 종료 기준 미통과 시 자동 수정 + 재검증 (대표님 개입 없이)
- **모호한 task → 검증 가능한 형태로**: "validation 추가" → "invalid input 테스트 작성 → 통과시키기"
- **케이스**: 5/7 SMS 사고 — sms_parser 코드는 4/19 완성됐지만 main 머지 안 돼서 18일 묵힘. 종료 기준 = "main에서 import 가능 + 운영 cron 통과"였으면 18일 사고 X

### A21~A24 vs superpowers:brainstorming 역할 분담

| 단계 | 도구 | 목적 |
|------|------|------|
| MODE 1 기획 | `superpowers:brainstorming` | 아이디어 → 스펙 정제, 무엇을 만들지 결정 |
| MODE 2 코드 직전 | **R-A21~A24 self-check** | 어떻게 안전하게 만들지 가드레일 (추측·과잉설계·인접 손대기·검증 누락 차단) |

- brainstorming 통과해도 코드 직전 R-A21~A24 self-check 1회 필수
- 둘은 보완재 — 중복 아님

<!-- id:R-A25 -->
### A25. 정기운영 게이트 면제 (2026-06-08 v2.1 신설)

- **MODE 4 정기운영은 MODE 1 게이트 면제**: 빡센점검(`haemilsia-rental-inspection`)·아이리스 동기화(`iris-excel-sync`)·파워링크 점검(`naver-powerlink-advisor`) 등 **검증된 반복작업**은 office-hours / CEO·ENG 리뷰 / Preflight / codex 게이트를 돌리지 않는다 (낭비 방지)
- **단, 결과 검증은 유지**: 점검 정합성·동기화 diff 등 작업 산출물 검증은 그대로 (게이트 면제 ≠ 검증 면제)
- **면제 대상 아님**: 신규 기능·신규 스킬·구조 변경은 정기운영이 아니므로 정상 게이트 적용
- **B14/B15 오인 방지**: 정기운영 중 Preflight·CEO/ENG 미실시는 위반(R-B14/R-B15)이 아니라 **정상** — tracker는 MODE 1 진입 시에만 체크

---

## B. 자주 실수 패턴 (Notion DB 이관 완료)

**이관일**: 2026-04-11 v1.4 → **최종 업데이트**: 2026-04-12 v2.0
**위치**: Notion DB [`⚠️ 규칙 위반 기록`](https://www.notion.so/6bb0c6c2ed9444baba4180ab70b35fb9) (`27c13aa7-9e91-49d3-bb30-0e81b38189e4`)
**범위**: **B1~B19** (2026-05-01 — B19 git push 누락 7일+ 신설)

**구조**: `위반코드` SELECT 필드 + `반복횟수` (수동 +1) + `재발방지` 개별 기록

**조회 방법**
- 세션 시작 TOP 5: `해결여부=false` + `반복횟수 DESC` + `limit 5` 쿼리
- 신규 위반 발생 시: 해당 `위반코드` row의 `반복횟수 +1` (A4 종료 루틴 7번에서 실행)
- 신규 패턴: Select 옵션에 추가 후 신규 row 생성

**REF v2.0 자동 집행 훅 현황** (2026-04-12 전체 활성화): 코드별 감지 시점·강도·훅·폴백은 아래 **D. 자동화 가드 맵** 단일 테이블 참조 — 2026-08-08 v2.3에서 중복 테이블 2개를 1개로 통합 (본 섹션 "하드코딩 목록 삭제" 원칙 적용).

**우회 방법**: 대표님 메시지에 `--force-B1` ~ `--force-B19` 형식으로 명시 → 훅이 우회 카운터 증가 (같은 코드 3회 이상 우회 시 Slack 알림 발송)

**원칙**: 개별 정의는 Notion DB가 단일 원본. rules.md 내 하드코딩 목록 삭제 (중복 관리 방지).

---

## D. 자동화 가드 맵 (B코드 ↔ 책임자)

> **목적**: B코드 위반 발생 시 5초 내 책임자 식별. 매주 또는 사고 시 갱신.
> **출처**: `docs/superpowers/specs/2026-04-25-claude-system-upgrade-v2-design.md` §4.3 (v0.2)
> **단일 테이블** (2026-08-08 v2.3): 구 B섹션 훅 테이블과 통합 — 감지·강도 컬럼 포함. 위반 이력 카운트는 Notion DB가 단일 원본 (표 내 스냅샷 컬럼 폐지).

| B코드 | 위반 내용 | 감지 · 강도 | 1차 가드 (자동) | 폴백 (수동) |
|------|----------|------------|----------------|-------------|
| R-B1 | 파일명 버전 누락 | PreToolUse:Write · hard_block | `check_filename_version.py` | 매니저 self-check |
| R-B2 | 세션 인수인계 미생성 | Stop · hard_block | `session-end-check.sh` + handoff-scribe (Sonnet) | 다음 SessionStart 미싱크 재시도 |
| R-B3 | 세션 시작 루틴 미실시 | Stop · hard_block | tracker `top5_queried` (SessionStart 훅) | 매니저 직접 호출 |
| R-B4 | 모델 추천 누락 (다른 모델 적합 시만 — 2026-07-07 완화) | Stop · soft_warn | UserPromptSubmit 훅 + tracker `tool_recommended` | 매니저 self-check |
| R-B5 | 스킬 설치 경로 오류 | PreToolUse:Write · hard_block | `check_skill_path.py` | 매니저 self-check |
| R-B6 | Notion 임의 저장 | Stop · soft_warn | tracker `notion_unauthorized` | 매니저 사전 컨펌 |
| R-B7 | 다운로드 파일명 패턴 | PreToolUse:Write · soft_warn | `check_filename_version.py --mode=B7` | 매니저 self-check |
| R-B8 | INTEGRATED.md 재빌드 누락 | Stop · hard_block | **`debounce_sync.sh`** (30s 디바운스) + tracker `pending_sync` | errors.log → SessionStart reminder |
| R-B9 | 스킬 설치 후 skill-guide 미등록 | Stop · hard_block | tracker `skills_dir + skill_guide` | 매니저 self-check |
| R-B10 | 메모리 상태 반영 누락 | Stop · hard_block | tracker `memory_updated` | `memory_patcher.py` 폴백 |
| R-B11 | 환경변수 토큰 노출 | PreToolUse(Bash/Write/Edit) · soft_warn | `check_token_exposure.py` | git-secrets pre-commit |
| R-B12 | ~~복습카드 미생성~~ **폐지** (2026-07-07) | — | — (훅 제거) | 수동 "복습해줘" 시만 |
| R-B13 | 에이전트 미dispatch | Stop · soft_warn | tracker `agent_dispatched` | 매니저 직접 dispatch |
| R-B14 | Preflight Gate 미실시 | Stop · hard_block | tracker `preflight_executed` (MODE 1 시) | 매니저 직접 (MODE 1 6번) |
| R-B15 | CEO/ENG 리뷰 미실시 | Stop · hard_block | tracker `ceo_eng_review_executed` (MODE 1 시) | 매니저 직접 dispatch (MODE 1 3·4번) |
| R-B16 | 세션 시작 에이전트 미dispatch | Stop · soft_warn | tracker `session_start_agents` | 매니저 직접 호출 |
| R-B17 | 세션 종료 에이전트 미dispatch | Stop · hard_block | tracker `session_end_agents` (handoff-scribe + notion-writer) | 매니저 명시 호출 |
| R-B18 | Agent dispatch 파일 경로 누락 | 수동 감시 | docs/agents.md SELF-CHECK (Wave 2 표준 템플릿) | 매니저 review |
| R-B19 | git push 누락 7일+ | SessionEnd/Stop · hard_block | **`session-end-git-sync.sh`** (1h 디바운스 자동 sync) | `session-start-git-push-check.sh` (7일+ 경고) |

> **B8 판정 주의**: `debounce_sync.sh`가 BUILD_SUCCESS를 남겼으면 위반 아님 (거짓 양성 다발 이력) — 아래 self-check 규칙 필수 적용.
>
> **B19 신설 배경 (2026-05-01)**: `~/.claude/` 12일치 변경 누적 미푸시 발견 — `build-integrated_v1.sh`가 INTEGRATED.md만 selective-add 하여 나머지 변경이 잔존했고, 종료 루틴에 git push 단계가 없었음. v2.0 자동화 가드로 해소.

### B8 self-check 규칙 (오기록 방지)

매니저가 handoff frontmatter `violations:` 작성 시 B8 판정 전 다음 체크:

```bash
# 마지막 시스템 문서 변경 후 30초 내 BUILD_SUCCESS 있는지 확인
tail -30 /tmp/claude-b8-debounce.log | grep BUILD_SUCCESS
```

→ BUILD_SUCCESS 항목 있으면 **B8 위반 아님** (debounce_sync.sh가 자동 처리). 매니저는 violations에서 B8 제외.

---

*Haemilsia AI operations | 2026.08.08 | v2.6 — 세션 tracker 워크트리 스코프 전환(`code/session_paths.sh` 신설, 훅 4개·에이전트 2개·문서 2개 경로 통일) + SESSION_RESUME 접기(연속 재시작 1줄, append-only) + v2.4 A5 모델 팔레트 Opus 5 갱신*