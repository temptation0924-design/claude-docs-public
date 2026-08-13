# session.md — 세션 루틴 + 기록 규칙

업데이트: 2026-08-12 | v5.6 — `session_paths.sh` 세션ID 포인터 보강 (ERR-68: 매니저 셸 전역 폴백 → 병행 세션 worklog 오취득·삭제 위험 차단, 회귀테스트 `code/test_session_paths_v1.sh` 20건) · 구 v5.5(2026-08-08): 세션 tracker 워크트리 단위 스코프 전환 + SESSION_RESUME 접기(연속 재시작 1줄) · 구 v5.3(2026-07-07): 훅 D/E 정식 배선 + handoff-guard 신설 + 모델 추천 전환(B4) + 복습카드 폐지(B12)

---

## 세션 시작

> **🤖 자동 처리 (SessionStart 훅)**: 세션 시작 시 시작 시각 자동 기록 (epoch + human time JSON) + 워크로그 초기화.
>
> 🆕 **워크트리 단위 스코프 (2026-08-08 전역오염 근본수정)**: 두 파일은 이제 `~/.claude/.session/<워크트리>-<해시>.start|.worklog`에 **워크트리별로 분리** 저장된다. 경로는 항상 아래로 해석 — **하드코딩 금지**.
> ```bash
> eval "$(bash ~/.claude/code/session_paths.sh --export)"   # → SESSION_START_FILE / WORKLOG_FILE / SESSION_SCOPE
> ```
> 키는 `CLAUDE_PROJECT_DIR`(훅에 워크트리 루트로 주입). 훅 stdin의 `.cwd`·셸 `$PWD`는 세션 중 `cd`를 따라가므로 키로 쓸 수 없다.
>
> 🆕 **세션ID 포인터 (2026-08-12 v2, ERR-68 근본수정)**: 위 08-08 수정은 **훅 경로만** 덮었다 — `CLAUDE_PROJECT_DIR`은 매니저·서브에이전트의 비대화형 셸엔 주입되지 않아(실측 UNSET) 그쪽은 조용히 전역 폴백했다. 이제 `CLAUDE_PROJECT_DIR`이 있을 때 `.session/by-id/<세션ID> → 슬러그` 포인터를 남기고, 없을 때는 `CLAUDE_CODE_SESSION_ID`(매니저·훅 양쪽 셸에 모두 존재)로 그 포인터를 되짚어 같은 스코프를 복원한다.
> - 포인터가 가리켜도 `<슬러그>.start`가 없으면 **기각**한다 (유령 스코프 차단)
> - 복원 실패 시에만 전역 폴백하며, `--export` 경로는 **stderr 경고**를 낸다. `source` 경로는 무음 — `session-tracker-log.sh`가 **PostToolUse**로 매 도구호출마다 부르므로 스팸이 된다
> - **cwd·git 추론은 쓰지 않는다**: cwd가 메인 리포로 새면 `claude-<해시>`라는 *실재하는 남의 스코프*를 조용히 집어들어 지금보다 나쁘다. 틀린 칸을 자신 있게 여느니 전역으로 떨어지고 시끄럽게 구는 쪽이 안전하다
> - 한 세션이 도중에 다른 워크트리로 옮기면 포인터도 따라 갱신된다 (내용 대조 후 변경 시에만 write — "파일 있으면 스킵"으로 최적화하면 옛 스코프에 박제되어 사고가 재현된다)
> - 회귀테스트: `bash ~/.claude/code/test_session_paths_v1.sh` (20건, 샌드박스 HOME 격리)
>
> **🆕 v2.0 SessionStart 훅 (2026-04-19 작성 → ⚠️ 2026-07-07에야 settings.json 정식 배선 — 그전엔 미실행 상태였음)**:
> - **훅 D** (`violation-prevention-inject.sh`): MEMORY.md TOP 3 위반 + 최근 handoff 미완료 키워드 매칭 → 해당 규칙 경고 주입 (2026-07-07 메모리 경로 버그 수정)
> - **훅 E** (`drift-detector.sh`): 최근 handoff `commits` 필드 ↔ git log 커밋 수 비교 → 차이 >2 시 🚨 표시 (⚠️ 타 리포 커밋은 오탐 가능 — 경고성)
> - **🆕 handoff 가드** (`session-start-handoff-guard.sh`, 2026-07-07 신설): ① 미싱크 handoff 파일명 나열 + [노션기록관] 재싱크 지시 주입 ② 최근 48h 트래커 중 작업 있었는데 handoff 없는 세션 탐지 → 소급 생성 지시 (B2 자동 보정)

### C+ 하이브리드 루틴 (매니저 직접 + Agent dispatch)

1. **매니저가 직접 병렬 도구 호출** (Stage 1, ~15초):
   - Notion TOP 5 쿼리 (notion-query-database-view 직접 호출)
   - MEMORY.md(핫) 스캔 (Read 직접) + 영속지식 검색은 `bash ~/.claude/code/recall.sh <키워드>` (콜드 MEMORY-knowledge.md 대상)
   - rules/session/skill-guide 핵심 로드 (Read 직접)
   - → 단순 작업은 Agent spawn 없이 **매니저가 직접** (spawn 오버헤드 0)
   - → Notion 지연 시: 1회 타임아웃 → 즉시 폴백 (캐시 참조). **이 폴백은 루틴상 정상 동작이며 B3(세션 시작 루틴 미실시) 위반이 아니다** — TOP5 조회를 *시도*했으므로 루틴은 실행된 것 (Notion 404/타임아웃은 외부 의존성 실패 R-A2 L4). `session-end-check.sh`가 `top5_queried != true`만 보고 emit하는 `⚠️ B3 ...(캐시 폴백 시 정상)` 경고는 **오탐** → 규칙감시관/핸드오프작성관이 제외. (2026-06-26 박제)
   - 🆕 **매일 첫 세션**: `[청소원 Sonnet]` Agent dispatch (환경 점검, 복잡 판단)
   - 🆕 **미싱크/drift handoffs 재시도**: 판정은 **즉석 구현 금지** — `python3 ~/.claude/code/handoff_sync_check_v1.py` 출력만 신뢰한다 (`UNSYNC`=CREATE 대기, `DRIFT`=싱크 후 편집됨·UPDATE 대기). `UNSYNC` 발견 시 `[노션기록관 Haiku]` dispatch → 사전 체크 로직(CREATE/UPDATE/SKIP) 자동 판정 + queue/ consume (최대 3회). `DRIFT`는 B2가 아니므로 내용 변경이 실제로 있을 때만 UPDATE.
     > ⚠️ **맨 `mtime > notion_synced_at` 비교 금지** (2026-08-09 오판 수정): 싱크는 항상 "① 노션 기록 → ② 로컬 frontmatter 패치" 순서라 **정상 싱크도 mtime이 `notion_synced_at`보다 늦다**(실측 99초, 119건 중 82건). 허용오차 0으로 재판정하면 이미 싱크된 파일을 영구 미싱크로 오판한다. 판정기는 600초 grace를 적용한다.

2. **매니저가 결과 병합 + 통합 응답 출력**:
   - TOP 5 표 (규칙감시관) + 관련 메모리 (기억관리관) + 지침 요약 (지침사서) + 환경 리포트 (청소원, 해당 시) + 환영 한 줄 (분위기메이커)
   - 고정 인사: "어떤 업무를 진행하세요? ☺️ 기획-실행-검증-운영모드 대기중입니다!"

3. **대표님 답변 → Stage 2 dispatch**:
   - `[모델추천관(tool-advisor) Haiku]` — 업무 설명 → 모델 등급 매칭 (rules.md A5, 2026-07-07 도구→모델 전환. 현재 모델과 다를 때만 1줄)
   - 매니저가 모드 라우팅 (MODE 1/2/3/4) + 스킬 매칭

> **수동 오버라이드**: "순차로 해" → 위 팀원 순서대로 실행. `/agent rule-watcher` → 단독 실행.
> **에스컬레이션**: 팀원 실패 시 Haiku→Sonnet→Opus 자동 승급 (agent.md 섹션 5 참조)
> **TOP 5는 Notion DB에서 동적 조회** (하드코딩 없음). `반복횟수` 필드가 자동 랭킹 소스.

### 세션 중 워크로그 기록 규칙

Claude Code(매니저)가 세션 중 다음 이벤트 발생 시 `$WORKLOG_FILE`(위 `session_paths.sh`로 해석)에 직접 append:

| 이벤트 | 기록 내용 | 방법 |
|--------|----------|------|
| MODE 전환 | `[HH:MM] MODE: MODE X → MODE Y 전환` | Bash append |
| **에러 발생** (신규) | `[HH:MM] ERROR: {에러 1줄 요약} \| {태그}` | Bash append |
| 에러 해결 완료 | `[HH:MM] ERROR_RESOLVED: {에러 요약}` | Bash append |

**에러 정의** (rules.md A2 참조):
- **L1 Exception/Crash**: tool 호출 실패, script 비정상 종료, hook block 등 즉시 가시
- **L2 진단 오류**: 표면 에러를 잘못된 원인으로 결론 비약 (이번 세션 노션기록관 사례)
- **L3 시스템 결함**: 핵심 파일/스크립트 버그 (이번 세션 memory_patcher 5버그 사례)
- **L4 외부 의존성 실패**: API 504, MCP 미응답, Notion 502 등 (단순 재시도로 복구 가능 시 제외)

→ L1~L3는 **반드시** worklog에 ERROR append + 세션 종료 시 노션기록관(2) 자동 dispatch.

> 🆕 **RESUME 접기 (2026-08-08)**: SessionStart가 재시작마다 `SESSION_RESUME`을 append하면 마커가 실제 내용을 압도해(실측 48분 10줄/내용 0줄) 핸드오프작성관이 무작업으로 오판한다. 직전 줄이 이미 RESUME이면 추가하지 않아 **연속 재시작은 항상 1줄로 접힌다**. 그래서 매니저가 위 표대로 MODE·ERROR를 append하는 것이 더 중요해졌다 — 내용 줄이 0이면 여전히 오판 대상이다.

**방어 로직**: 파일 없으면 자동 생성 후 append.
```bash
eval "$(bash ~/.claude/code/session_paths.sh --export)"
[ ! -f "$WORKLOG_FILE" ] && echo "[$(date +%H:%M)] SESSION_START: (auto-created)" > "$WORKLOG_FILE"
echo "[$(date +%H:%M)] MODE: MODE 1 → MODE 2 전환" >> "$WORKLOG_FILE"
echo "[$(date +%H:%M)] ERROR: 노션기록관 큐 재시도 오진단 | MCP,Notion" >> "$WORKLOG_FILE"
```

---

## 작업 단위 루틴

> ⚠️ **복습 카드 자동 트리거 폐지 (2026-07-07 대표님 결정, B12 폐지)** — 자동 생성·자동 발송 없음.
- **수동 호출만 유지**: "복습해줘" / "정리해줘" → `study-coach` 단독 dispatch (`#claude-study` 발송도 이때만)
- 구 상세 스펙: [`docs/rules/task-routine.md`](docs/rules/task-routine.md) (참고용 보존)

---

## 세션 종료

> 🆕 **Workflow 오케스트레이션 옵션 (2026-07-07 대표님 옵트인)**: 아래 Stage 1~2 전체를 `Workflow({scriptPath: "~/.claude/workflows/session-end.workflow.js"})` 호출 한 번으로 실행 가능 — Stage 1 병렬 → Stage 2 순차가 스크립트로 결정적 보장 + 진행상황 트리 표시. 수동 다중 Agent dispatch는 폴백 경로.
>
> 🆕 **조기 폴백 게이트 (2026-07-19 신설 — 2026-07-17 4시간 stall 사고 재발방지)**: session-end-cplus workflow는 **1회만 호출**한다. Stage 1 결과가 stall/실패면(에이전트 반환 null, 또는 호출 후 **5분 내 완료 통지 없음**) 매니저는 **workflow를 재호출하지 말고 즉시 개별 Agent 순차 dispatch로 폴백**한다(규칙감시관 → 핸드오프작성관 → 노션기록관 → 슬랙배달관).
> - **근본원인(2026-07-17 규명, 트랜스크립트 18개 전수분석)**: 6/6 stall은 코드 버그가 아니라 **워크플로우 서브에이전트의 추론(inference) 호출 stall** — tool_result 수신 후 다음 모델 응답이 445~1,956초 무응답(18개 전부 동일 패턴, tool 실행은 +0초 정상). harness가 180초 무진행마다 자동 재시도(각 6회=18회)하며 4시간 소모. 재시도는 같은 인프라 stall을 못 풀어 **무의미** → 조기 폴백이 유일 해법. 개별 Agent는 매니저가 직접 감시하므로 stall 시 즉시 인지·중단 가능.
>
> 🆕 **야간 안전망 (2026-07-07)**: 세션 종료 시 Notion 싱크가 누락/실패해도 launchd `com.haemilsia.notion-handoff-sync`(매일 03:17)가 미싱크 handoff를 일괄 CREATE 싱크 (env-info.md 「로컬 자동화」 참조).

### C+ 병렬 dispatch 루틴

1. **자체 점검**: 오늘 TOP 5 패턴 중 어긴 것 확인 (매니저가 직접 판단)
2. **Stage 1 — 매니저가 필수 1명 + 조건부 1명 dispatch**:
   - `[핸드오프작성관 Sonnet]` — `$WORKLOG_FILE` 참조 → `~/.claude/handoffs/세션인수인계_YYYYMMDD_N차_v1.md` 생성 (frontmatter 포함) → `$WORKLOG_FILE` 삭제
     - ⚠️ **dispatch 프롬프트에 `SESSION_START_FILE`·`WORKLOG_FILE` 절대경로를 반드시 명시**한다 (매니저가 `session_paths.sh --export`로 해석해 전달). 서브에이전트가 스스로 추론하면 워크트리 경계를 넘어 엉뚱한 파일을 읽고 지운다 — 2026-08-08 근본수정
       > 🔴 **이건 권고가 아니다**: 2026-08-12 ERR-68에서 **매니저 셸 자체가** 전역 폴백해, 병행 세션이 4시간 동안 쌓은 worklog 20건을 제 것으로 기재하고 루틴 끝에 삭제할 뻔했다(사전 발견·미발생). v5.6 포인터 복원이 1차 방어선이고, 절대경로 명시는 그게 실패했을 때의 2차 방어선이다. 둘 다 지킬 것.
   - `[노션기록관 Haiku(2)]` — ⚡ **자동 트리거**: `$WORKLOG_FILE`에 `ERROR:` 라인 1건 이상 → 강제 dispatch (skip 금지). 0건이면 스킵
   - ~~`[복습카드관 Opus]`~~ — **폐지 (2026-07-07, B12)** — 수동 "복습해줘" 요청 시에만
   - → 예상 소요: **5~8초**

3. **Stage 2 — 매니저가 결과 병합 후 3명 dispatch** (순차, Stage 1 결과 필요):
   - `[노션기록관 Haiku]` — handoffs/ frontmatter 파싱 → 사전 체크 로직(CREATE/UPDATE/SKIP) → Notion 작업기록 DB 싱크 → `notion_synced: true` + `notion_page_id` + `notion_synced_at` 3필드 갱신
   - `[규칙감시관 Haiku]` — TOP 5 자체점검 + **실제 위반만** DB update (반복횟수 +1). **B3 판정 권한 보유** — 세션 시작 TOP5 조회가 캐시 폴백된 경우(Notion 404)는 B3 아님으로 확정. 핸드오프작성관은 이 확정을 따라 tracker의 `B3 ...(캐시 폴백 시 정상)` 항목을 frontmatter에서 제외
     > 🔴 **Stage 2에 있는 이유 — B2 과대계상 근본수정 (2026-08-13 대표님 승인)**: 감시관이 Stage 1에서 핸드오프작성관과 병렬로 돌면 **채점 시점에 핸드오프 파일도 Notion 작업기록도 아직 없다**. 그래서 정상 종료 세션도 매번 B2(인수인계 미생성·작업기록 DB 미저장)로 찍혀 반복횟수가 부풀었다(08-13 기준 37회, 대부분 오탐 의심 — 2026-08-12 6차에서 제기·보류). **판정은 채점 대상이 만들어진 뒤에 한다**. 감시관은 핸드오프 경로·싱크 결과를 증거로 받아, 파일이 실제로 없거나 싱크가 실제로 실패했을 때만 B2를 기록한다. tracker에 남은 B2 경고는 루틴 완료 *전* 시점 기준이라 항상 남으므로 그것만 보고 판정하지 않는다.
   - `[슬랙배달관 Haiku]` — #general-mode 작업일지 (#claude-study 학습 카드 자동 발송은 2026-07-07 폐지 — 수동 복습 요청 시만). 감시관 판정을 본문에 싣으므로 감시관 뒤에 온다
   - → 예상 소요: **5~8초**

> **2026-04-12 간소화**: 청소원 세션 종료 dispatch 제거 (매일 첫 세션 시작에서만 실행). 노션기록관(2)·복습카드관은 조건부 실행으로 전환.

4. **매니저가 최종 요약 보고**:
   - 세션 통계 (완료 작업 수, 소요시간, 복습 카드 수)
   - 소요시간: `eval "$(bash ~/.claude/code/session_paths.sh --export)"; echo $(( ($(date +%s) - $(jq -r '.epoch' "$SESSION_START_FILE")) / 60 ))분`

> **B2 위반 방지**: 핸드오프작성관이 Stage 1에 필수 포함 → 시스템 구조상 인수인계 누락 불가
> **수동 오버라이드**: "순차로 해" → 위 팀원 순서대로 실행
> **에스컬레이션**: 팀원 실패 시 자동 승급 (agent.md 섹션 5 참조)

- **상황판 조건부 갱신**: 그 세션 대화에서 Claude가 서재 DB·훈련장 페이지에 MCP 쓰기를 실행한 경우에만 `haemilsia-stock-dashboard`(서재 DB `b0cef632-…`) 절차로 1회 재발행 — **지면 3종을 일지 원장 → 변경분 종목 등기부 → 상황판 순서로**(하위 URL이 먼저 확정돼야 허브가 링크를 실음). 논블로킹 — 실패·2분 초과 시 조용히 스킵하고 보고만, 세션 종료를 차단하지 않는다.

### 🆕 v2.0 MEMORY 동시 패치 (2026-04-19, 2026-07-27 동시쓰기 사고 후 개정)

**핸드오프작성관 Stage 1 확장 동작**:
1. **차수 원자 예약**: `bash ~/.claude/code/handoff_slot.sh $(date +%Y%m%d)` → 반환된 스텁 파일에 핸드오프 작성 (`$WORKLOG_FILE` 참조). 차수 수동 계산·dispatch 프롬프트의 차수 지정은 무효 — 스크립트 결과가 유일 진실 (같은 날 차수 충돌 2연속 재발 근본수정)
2. **`memory_patcher.py` 실행 — v2.3부터 mkdir-lock(`~/.claude/.memory.lock.d`) 자체 획득**:
   - MEMORY.md 🟢 최근 완료 / 🔴 할 일 / ⚡ 반복 위반 TOP 3 동시 갱신
   - ⚠️ `with_lock` 래핑 금지 (이중 락 대기 유발). 구조 손상 감지(exit 4) 시 git 이력 복구 후 재시도
3. **실패 시 queue 저장** (`~/.claude/queue/pending_memory_*.json`) — 세션 종료 **차단 없음** (B2 방지)
4. **MEMORY.md(핫) 용량 점검** (rules.md A9, 2026-07-07 바이트 기준 개정): `wc -c` > 8,000바이트 시 **그 자리에서 압축**(7일 롤링 제거 → 장문 줄 압축 → archive mv). 영속지식은 콜드 `MEMORY-knowledge.md`로 분리됨(2026-06-26 v2.5) — 새 feedback/project/reference 인덱스 줄은 콜드에 추가, 핫엔 추가 금지.
   - **압축 편집 동시성 규칙 (2026-07-27)**: MEMORY.md 압축·수동 편집은 Write/Edit 도구로만 (bash sed/리다이렉션 금지 — memory-write-guard 훅이 락 경합을 막아줌). 훅이 거부하면 2~3초 후 **재-Read → 최신 내용 위에** 재편집. 병렬 세션 존재 가능성을 항상 전제할 것
5. **daily 스크래치패드 검토**: 세션 중 임시메모는 `bash ~/.claude/code/scratch.sh "메모"`로 적립. 종료 시 `memory/daily/오늘.md` 검토 → 영속 가치만 정식 카드로 승격(+콜드 인덱스), 나머지 폐기.

**노션기록관 Stage 2 확장 동작**:
1. handoff frontmatter → Notion DB 메타 저장 (기존)
2. **handoff 본문 → page child blocks append (신규)** — 2000자 분할, 3 req/sec
3. 전체 성공 → `notion_synced: true` / 일부 실패 → `notion_synced: partial` + `blocks_created: N`

---

## 노션 기록 원칙
→ 상세 스펙은 [`docs/rules/notion-logging.md`](docs/rules/notion-logging.md) 참조 (DB 자동 판단 표 + 기록 형식 표)
- 핵심: 저장 전 2단계 확인 (`rules.md` A2 참조) + 임의 저장 금지

---

## 오류 발생 시
→ 상세 스펙은 [`docs/rules/error-handling.md`](docs/rules/error-handling.md) 참조 (감지 키워드 + 기록 형식 + 절차 6단계)
- 핵심 흐름: 에러로그 DB 먼저 검색 → task 체크리스트 → 원인 분석 → 재발 방지 → **복습 카드 자동 트리거** ([`docs/rules/task-routine.md`](docs/rules/task-routine.md) 참조)

---

