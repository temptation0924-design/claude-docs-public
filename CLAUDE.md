# CLAUDE.md — Haemilsia AI operations

**버전**: v4.7 | **업데이트**: 2026-08-08
**적용**: Claude.ai (웹) + Claude Code (터미널) 통합

> **CLAUDE.md = 라우팅 허브**. 모든 실행 원칙은 모드/스킬/루틴으로 이관 완료.
> - 하위원칙 + 자주 실수 패턴 → [`rules.md`](rules.md)
> - 세션 시작/종료 루틴 → [`session.md`](session.md)
> - 스킬 관련 규칙 (1% 룰 포함) → [`skill-guide.md`](skill-guide.md)
> - 상세 실행 절차 → 각 MODE 워크플로우

---

## 0. ID 체계 (2026-04-22 신설)

시스템 지침 조항에 고유 ID가 박제되어 있음. 대표님·AI·DB가 같은 주소로 같은 규칙을 지칭 가능.

- **CLAUDE.md**: `C-01 ~ C-36` (이 파일)
- **rules.md A섹션**: `R-A1 ~ R-A25` (하위원칙)
- **rules.md B섹션**: `R-B1 ~ R-B19` (위반 코드, Notion DB 단일 원본)

**사용 예시**:
- 대표님: "그 **C-14** 수정해줘" → AI가 정확히 MODE 1 9번(대표님 승인) 지칭
- AI: "**R-B4** 위반 1회 기록합니다" → 규칙위반 DB에 정식 ID 박제
- 조회: `~/.claude/code/id-lookup_v1.sh R-A4` → 해당 항목 전문 출력
- 레지스트리: `env-info.md ## 📇 ID 레지스트리`

"B4", "MODE 1-7" 같은 자연어 참조도 계속 유효 — ID는 정밀도 보완재.

---

## 1. 개요

<!-- id:C-01 -->
### 사람 역할
| 누가 | 하는 것 | 안 하는 것 |
|------|---------|-----------|
| 이현우 대표님 | 기획 + 업무 계획 수립 + 도구 선택 승인 + 결과 확인 | 코드 직접 작성, 배포, 오류 수정 |

<!-- id:C-02 -->
### 도구 계층
| 도구 | 계층 | 역할 |
|------|------|------|
| Claude Code | **마스터** (기본값) | 코드 작성/수정, 배포, Git push, 터미널 실행, 스킬 관리, 자율 실행 |
| 브라우저 자동화 | **마스터 직속** | Claude Code가 직접 제어 ─ **Claude_in_Chrome**(대표님 로그인된 크롬 그대로: 네이버 광고·검수 등 로그인 필요 작업) · **gstack `browse`·`scrape`**(깨끗한 브라우저 QA·자료추출) |
| Claude.ai | 보조 | Notion·Slack·Figma MCP 연동, 시각화, 문서 생성, 웹 검색. **업무 기획 + 계획 수립 전담** |
| Cowork | 보조 | MCP 없는 사이트를 **대표님이 직접** 클릭, 모니터링, 로컬 파일 편집 |

> **모델 추천**은 **MODE 1 10번** (계획 기반) 또는 **session.md 세션 시작 3번** (단순 업무)에서 자동 실행 — 현재 모델과 다른 모델이 적합할 때만 1줄 명시 (rules.md A5, 2026-07-07 도구 추천에서 전환). 도구(Code/Claude.ai/Cowork) 구분은 위 표 참조.

<!-- id:C-03 -->
### 지침 읽기 체계
| 도구 | 지침 읽는 곳 |
|------|------------|
| Claude Code | `~/.claude/CLAUDE.md` (Git repo — **원본**) |
| Cowork | `~/.claude/CLAUDE.md` (Git repo — **원본**) |
| Claude.ai | **GitHub raw URL 통합본** — `https://raw.githubusercontent.com/temptation0924-design/claude-docs-public/main/INTEGRATED.md` (8개 md 자동 concat, 5분 캐시) |

> **리포 2개 체제 (2026-08-09 분리)**: 원본 `claude-system-docs`는 **private** — 핸드오프·메모리·업무기록이 들어 있어 공개 금지. Claude.ai가 읽는 `claude-docs-public`은 **공개 미러**로, 지침 md 12종만 담는다(`INTEGRATED.md`·`CLAUDE-core_v1.md`·개별 7종·`on-demand/` 3종). 미러에 업무 기록을 넣지 말 것. 경위: 메모리 `project-public-repo-privacy-exposure`.

> **원본**: Git 리포지토리(`~/.claude/`)가 유일한 원본. 수정 시 → Git 파일 먼저 수정 → `build-integrated_v1.sh --push`로 공개 미러 재빌드·푸시 (~10초). Notion 개별 백업 7페이지는 2026-04-12 폐기 (비효율). Notion은 DB 기록 전용 (작업기록/에러로그/규칙위반).

---

<!-- id:C-04 -->
## 2. 파일 라우팅 맵

| 트리거 | 읽을 파일 | 역할 |
|--------|----------|------|
| "세션", "시작", "마무리" | `session.md` | 세션 시작/종료 루틴 |
| 규칙 위반 발생 시 | `rules.md` | 하위원칙 + 자주 실수 패턴 |
| 스킬 확인/추천 | `skill-guide.md` | 스킬 목록 + 추천 규칙 |
| 환경/DB ID/API | `env-info.md` | 환경, MCP, Notion ID, 배포 인프라 |
| "에이전트", "agent", "팀 에이전트" | `agent.md` | 에이전트 레지스트리 조회 |
| "기획", "계획", "plan", "만들자", "아이디어" | MODE 1 워크플로우 | 기획 모드 진입 |
| "진행해", "실행", "OK" | MODE 2 워크플로우 | 실행 모드 진입 |
| "검증해줘", "점검해줘", "체크해줘" (MODE 1 컨텍스트) | MODE 1 내 Preflight | 기획 중 **계획** 사전검증 (3 Agent 게이트) |
| "검증해줘", "점검해줘", "체크해줘" (MODE 3 컨텍스트) | MODE 3 워크플로우 | 실행 후 **코드** 사후검증 (/qa + /review) |
| "테스트해줘", "QA해줘", "배포 확인" | MODE 3 워크플로우 | 실행 후 품질 검증 |
| "업무하자" | MODE 1~4 선택 질문 | 모드 선택 후 진입 |
| "quick", "빠르게", "간단히" | /gsd-quick | 간소화 모드 |
| "설명해줘", "쉽게 풀어줘", "쉽게 설명해줘", "비유로 설명", "무슨 말이야?", "다시 설명" | `briefing.md` | 쉬운 설명 브리핑 (수동 재설명) |
| "슬랙", "slack", "채널", "브리핑 채널" | `slack.md` | 슬랙 운영 허브 (채널 지도 + 로드맵) |
| 항상 (기본) | `CLAUDE.md` | 이 지침의 로컬 버전 |

---

## 3. 업무 모드 시스템

> **C+ 에이전트 시스템**: 모든 MODE 루틴은 `agent.md` v2.2의 19명 전문 팀원을 통해 병렬 dispatch됩니다. 세부 트리거는 `agent.md` 섹션 3 참조. 에이전트 프로필은 `~/.claude/agents/` 디렉토리 참조. CEO+ENG 리뷰는 **병렬 실행**.

모든 업무는 4가지 모드 중 하나로 자동 라우팅된다.

<!-- id:C-05 -->
### MODE 1: 기획 모드 (Planning)
**트리거**: "아이디어 있어", "이거 만들자", "계획 세워보자", "plan", "기획해줘", "기획하자", "계획하자"

**워크플로우**:
<!-- id:C-06 -->
1. `/office-hours` — 아이디어 검증 (소크라테스 질문)
<!-- id:C-07 -->
2. `superpowers:brainstorming` — 설계 정제 + 스펙 문서 작성
<!-- id:C-08 -->
3. `/plan-ceo-review` — 전략적 관점 리뷰
<!-- id:C-09 -->
4. `/plan-eng-review` — 아키텍처 관점 리뷰
<!-- id:C-10 -->
5. `superpowers:writing-plans` — micro-task 분해 (2~5분 단위, 묻지 말고 전부 분해)
<!-- id:C-11 -->
6. **Preflight Gate** (자동) — 5번 완료 후 자동 실행, 대표님 트리거 불필요
   - 3 Agent 사전검증 → 90% 이상 PASS → 7번으로
   - 90% 미만 FAIL → 자동 수정 → 재검증 반복 (PASS까지)
<!-- id:C-12 -->
7. **📘 계획 이해 브리핑** — Preflight PASS 직후 자동 실행 → `briefing.md` §2-3 풀버전 포맷 적용 (큰 그림 1줄 / 비유 / 결과물·시간·의존성 / "궁금한 거 있으세요?")
<!-- id:C-13 -->
8. **🤖 /codex consult 게이트** (기획 승인 직전, 외부 AI 세컨드 오피니언)
   - 조건: task 수 **≥ 20 → 자동 실행** / 미만 → opt-in (대표님 요청 시에만)
   - 동작: `/codex consult` → Codex가 계획 검토 → P1 Critical / P2 Warnings 피드백 → 수정 라운드 (PASS까지 반복)
   - PASS 기준: Codex GATE PASS 또는 대표님 "무시 진행" 판단
<!-- id:C-14 -->
9. 대표님 승인
<!-- id:C-15 -->
10. **🎯 모델 추천 + 스킬 매칭** — 승인된 계획 기반 자동 실행 (2026-07-07 도구 추천에서 전환)
    - 모델 추천: "이 작업은 **[모델]** 권장 (이유: ~)" — 현재 모델과 다를 때만 1줄. 팔레트: Haiku 4.5(단순·반복·대량) / Sonnet 5(일반) / Opus 5(복잡 코딩·리뷰) / Fable 5(기획·고위험) — 상세 rules.md A5
    - 스킬 매칭: `skill-guide.md` 키워드 매칭 → 1%라도 맞으면 invoke
    - 매칭 스킬 없음 → MODE 2 완료 후 자동 스킬화 대상으로 플래그
    - → MODE 2로 전환

> 간단한 기획: `/gsd-quick` → full 워크플로우 스킵

<!-- id:C-16 -->
### MODE 2: 실행 모드 (Execution)
**트리거**: "OK!", "진행해", "끝까지 해줘"

**워크플로우**:
<!-- id:C-17 -->
1. fresh context 확보 (GSD 원칙 — 긴 작업 시 task별 새 context)
<!-- id:C-18 -->
2. `superpowers:subagent-driven-development` — task별 별도 에이전트 (묻지 말고 전부 실행)
<!-- id:C-19 -->
3. `superpowers:test-driven-development` — 코드 작업 시 TDD 강제
<!-- id:C-20 -->
4. 2단계 코드리뷰 — spec 준수 + 코드 품질
<!-- id:C-21 -->
5. **🤖 /codex review 게이트** (코드 완료 후, 외부 AI 코드 리뷰)
   - 조건: task 수 **≥ 20 → 자동 실행** / 미만 → opt-in (대표님 요청 시에만)
   - 동작: `/codex review` → diff/PR 대상 외부 AI 코드 리뷰 → 버그/보안/품질 이슈 → 수정 라운드 (PASS까지 반복)
   - PASS 기준: Codex GATE PASS
<!-- id:C-22 -->
6. `/ship` 또는 `/land-and-deploy` — 배포 (해당 시)
<!-- id:C-23 -->
7. **🎁 자동 스킬화 제안** — MODE 1 10번에서 매칭 스킬이 없었던 경우 자동 실행
   - "이 작업을 스킬로 만들까요?" 질문
   - 승인 시 → `skill-manager` 스킬로 자동 생성
   - → `skill-guide.md` 자동 등록 (로컬 + Notion 양쪽)
   - 재사용 불가능한 일회성 작업이면 스킵

> 간단한 실행: `/gsd-quick "작업 내용"`

<!-- id:C-24 -->
### MODE 3: 검증 모드 (Quality)
**트리거**: 배포 후 자동, "테스트해줘", "QA해줘", "배포 확인"

**워크플로우**:
<!-- id:C-25 -->
1. `/qa` — 자동 QA 테스트
<!-- id:C-26 -->
2. `/review` — 코드 리뷰
<!-- id:C-27 -->
3. `/canary` — 배포 후 모니터링
<!-- id:C-28 -->
4. `/cso` — 보안 감사 (필요 시)
<!-- id:C-29 -->
5. `/retro` — 프로젝트 완료 후 회고 (필수)

<!-- id:C-30 -->
### MODE 4: 운영 모드 (Operations)
**트리거**: 세션 시작/종료, 일상 업무

**워크플로우**:
- 세션 시작 → `session.md` 루틴
- 일상 업무 → `skill-guide.md` 키워드 매칭
- 세션 종료 → `session.md` "세션 종료" 루틴 (핸드오프작성관 → `handoffs/세션인수인계_YYYYMMDD_N차_v1.md` 자동 생성 + Notion 기록)

<!-- id:C-31 -->
### 전역 브리핑 레이어 (Easy Briefing)

모든 MODE 공통 — 대표님 요청당 1회 착수 전 쉬운 설명 자동 발동. 복잡도 적응형(원라이너 / 3줄 / 풀버전). 연속 작업·마이크로 요청은 스킵. 상세는 `briefing.md` 참조.

- MODE 1 기획 진입 시: **풀버전** (기존 7번 본문)
- MODE 2·3·4 새 요청: **원라이너** 또는 **3줄**
- 수동 재설명 키워드 (`"설명해줘"`, `"쉽게 설명해줘"` 등 6종) 수신 시: **3줄 이상** 재설명
- 대화형 질문도 **스킵하지 않음** — 원라이너로 찍고 답변

<!-- id:C-36 -->
### 전역 코드 가드레일 레이어 (Karpathy R-A21~A24)

모든 MODE 공통 — **코드 작성 착수 직전 self-check 1회 필수**. Andrej Karpathy 4원칙을 부동산/운영 맥락으로 박제. 상세·케이스는 [`rules.md`](rules.md) R-A21~A24 참조.

| ID | 원칙 | 한 줄 가드레일 |
|----|------|---------------|
| **R-A21** | Think Before Coding | 추측 금지 — 모르면 멈추고 질문, 트레이드오프·가정 명시 |
| **R-A22** | Simplicity First | 요구된 것만 — 일회성 추상화·YAGNI 금지 |
| **R-A23** | Surgical Changes | 건드릴 것만 — 인접 코드 "개선"·dead code 손대기 금지 |
| **R-A24** | Goal-Driven Execution | 검증 가능한 종료 기준 정의 → PASS까지 루프 |

> `superpowers:brainstorming`(MODE 1, 무엇을 만들지)과 **보완재**. brainstorming 통과해도 코드 직전 self-check 별도 1회. MODE 2 진입 시 자동 발동.

### 모드 전환 규칙
<!-- id:C-32 -->
- **"업무하자"**: MODE 1~4 중 어떤 모드로 진행할지 질문 → 대표님 선택 후 해당 모드 진입
<!-- id:C-33 -->
- **기획 → 실행**: 대표님 "OK!" 또는 90% 검증 통과
<!-- id:C-34 -->
- **실행 → 검증**: 작업 완료 또는 배포 후 자동
<!-- id:C-35 -->
- **어디서든 → 기획**: 대표님 "계획 세워보자", "기획해줘" 트리거

---

*Haemilsia AI operations | 2026.08.08 | v4.7 — MODE 1 10번 모델 팔레트 Opus 4.8 → Opus 5 갱신 (rules.md R-A5 v2.4와 동기화, 상세·폴백은 A5 단일 원본)*
