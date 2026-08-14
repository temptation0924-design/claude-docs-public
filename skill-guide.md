# skill-guide.md — 모드 기반 스킬 가이드 v3.0

**업데이트**: 2026-08-08
**카테고리**: 11개 (업무 영역 기준) + 모드별 핵심 스킬
**등록 스킬**: 로컬 `~/.claude/skills/` 178개 디렉토리 (Haemilsia 커스텀 + gstack + GSD 67 + 외부 설치) + Superpowers/슬랙 등 플러그인

> ⚡ **컨텍스트 다이어트 (2026-07-07 신설 · 2026-08-08 GSD 1.42 재정렬)**: 저사용 스킬 **65개**(gsd-* 54 — 핵심 7종 quick/help/execute-phase/plan-phase/pause-work/resume-work/progress 및 네임스페이스 라우터 6종 gsd-ns-* 제외 · ios-* 5 · supanova 5 · benchmark-models)를 settings.json `skillOverrides: "name-only"` 처리 — 시스템 프롬프트에 이름만 실리고 설명은 생략됨 (매 턴 토큰 절감). **트리거 매칭은 이 파일이 담당하므로 발동엔 지장 없음** (이름으로 invoke 가능). 특정 스킬을 자주 쓰게 되면 settings.json에서 해당 항목만 삭제해 복원.
>
> 🔁 **GSD 1.42 명령 통합 (2026-08-08 업그레이드)**: 마이크로 스킬 24종이 삭제·플래그로 흡수됨 — `/gsd-note`→`gsd-capture --note` · `/gsd-do`→`gsd-progress --do` · `/gsd-add-phase` 등→`gsd-phase` · 워크스페이스 3종→`gsd-workspace`. 신규: `gsd-sketch`(HTML 목업) · `gsd-spike`(기술검증) · `gsd-spec-phase`(요구사항 정제) · `gsd-plan-review-convergence`(교차 AI 계획 수렴) · `gsd-graphify`(지식 그래프). 옛 이름 호출 시 새 명령으로 라우팅할 것.

---

## 사용 규칙

1. 작업 시작 전 이 파일 읽기
2. **모드 확인** → 현재 작업이 어떤 모드인지 판별
3. 키워드 매칭 → 해당 스킬 SKILL.md 읽기
4. **1% 룰 (Superpowers 원칙)** → 관련 스킬이 1%라도 해당되면 invoke하여 읽는다.
   - "아마 안 맞을 것 같다"는 건너뛰는 이유가 **아니다**.
   - 스킬이 실제로 불필요하다고 확인된 후에만 스킵 가능.
5. 스킬 2개 이상 해당 시 모두 읽기
6. 새 스킬 생성 시 이 파일에 등록

---

## 모드별 핵심 스킬 (자동 호출)

> 각 모드 핵심 스킬 요약. 상세 트리거/설명은 아래 카테고리 1~10 참조. 트리거 상세 검색은 `skill-manager` 스킬.

### MODE 1: 기획 (8개)
- `office-hours` / `brainstorming` / `writing-plans` — 아이디어 → 설계 → 분해 (자동)
- `plan-ceo-review` / `plan-eng-review` / `plan-design-review` — 병렬 리뷰
- `preflight-check` / `autoplan` — 자동 사전검증 + 결정 위임

### MODE 2: 실행 (7개)
- `subagent-driven-dev` / `test-driven-dev` / `executing-plans` — 자동 실행 (Superpowers)
- `gsd-quick` / `gsd-execute-phase` — GSD 간소/정식 실행
- `ship` / `land-and-deploy` — 배포 (로컬/프로덕션)

### MODE 3: 검증 (8개)
- `qa` / `qa-only` / `review` / `investigate` — 테스트·리뷰·디버그
- `canary` / `benchmark` — 배포 후 모니터링/성능
- `cso` / `retro` — 보안 감사 / 회고

### MODE 4: 운영 (7개)
- `system-docs-sync` / `skill-manager` — 시스템 문서/스킬 관리
- `haemilsia-rental-inspection` — 임대점검 (간편/빡센)
- `iris-excel-sync` — 아이리스공실 엑셀 → Notion DB 동기화
- `gsd-pause-work` / `gsd-resume-work` — 세션 종료/재개
- `careful` / `freeze` / `guard` — 프로덕션 보호

---

## ⭐ 이현우 대표님 제작 스킬 (최우선)

> 대표님 직접 제작 스킬. 최우선 참조. (기존 카테고리 1~10에도 중복 표시)

### 🏢 해밀시아 (9개)
- `haemilsia-rental-inspection` — 임대점검, 일일점검, DB점검, 점검보고서, 검증해줘
- `iris-excel-sync` — 아이리스공실 엑셀 이미지 → Notion DB 동기화 (diff·매핑·확인 게이트·보정) — v2.3 (2026-08-14): 퇴거일 컬럼 "완료" 단독=일반임대 가능 매핑 추가
- `haemilsia-bot-dev` — 해밀봇 기능 추가, 명령어 추가, Block Kit, 드릴다운
- `haemilsia-bot-deploy` — 봇 배포, Railway 배포, 환경변수 수정
- `railway-notion-connect` — Railway↔Notion 연동, 503/401/404 디버깅
- `haemilsia-property-card` — 부동산 수익카드, 매매/대환 분석, 카톡PNG
- `haemilsia58` — Quiet Luxury 홈페이지 제작, 해밀시아58 스타일로, 프리미엄 스튜디오/펜션/공간 대여 (React 19 + Tailwind 4)
- `legal-court-agent` — 🏛️ 법률소송 1차 대응 7 에이전트 시스템(Wave 1 자료조사 3 + Wave 2 판사+양측+제3자 4 + Wave 3 최종중재 1). 트리거: "법률검토", "법률 자문", "사건 분석", "/법률검토", "소송 분석". 워크플로우: ~/legal-court-agent/WORKFLOW_v1.md (2026-05-25 신설)

**💡 임대점검 2중 체계**: 간편(v1.0, Railway 07:30 자동) + 빡센(v2.0, 29항목 수동)

### 🤖 자동화 (5개)
- `taeheung-image-gen` — 🖼️ 태흥 블로그 이미지 근거 생성 (3층 체계: 실사 주력 / 나노바나나 Pro 제품컷 / Soul 2.0 사용장면 카테고리당 1~2장). 브리프 접지 → 힉스필드 생성 → 2축 검수(fit·topic ≥3.0) → zgen_ 입고. 트리거: "생성해줘"(태흥 문맥)/"사용장면 만들어줘"/"이미지 풀 생성으로 채워줘"/LOW_STOCK·low_score 경보 후. 실사 승격은 README.taeheung §4 별도 경로. **생성 전 프라이버시 재판정 먼저** — 실사가 privacy로 묶여 있으면 `promote_photos --dry-run --unmask`가 오탐을 회수한다(08-13: 291장 중 273장 회수) (2026-08-04 신설 / 08-13 갱신)
- `slack-info-briefing-builder` — 슬랙 브리핑, RSS 봇
- `landing-page-deploy` — 랜딩페이지 Netlify + Notion 연동
- `pwa-spawn` — 🍳 요리책 PWA 패턴(YouTube URL → AI 분석 → 큐레이션 라이브러리) 도메인 복제. Step 0 supanova-design-engine 필수. AI 공부/부동산 임장/운동 영상 정리 등 spawn (2026-05-18 신설)
- `blog-keyword-monthly` — 📝 블로그 월간 키워드 자동 선정. 태흥스펀지 파워링크 노출수 상위 → 블로그 1페이지 우리글(haemilsia2277) 없는 키워드만 → 그 달 월·목 전체 배정 → 해밀시아블로그DB 기록. 트리거: "블로그 키워드 뽑아줘"/"이번달 블로그 키워드"/"월간 키워드". 월간 리마인더 cron 연동 (2026-06-26 신설)

### 📋 시스템 (3개)
- `system-docs-sync` — 시스템 문서 수정
- `skill-manager` — 스킬 관리 (목록/검색/추가/삭제)
- `file-organizer` — 파일 정리, 다운로드 정돈

### 🎨 개인화 (7개)
- `screenshot-check` — 스크린샷 확인
- `petitlynn-color` — 쁘띠린 색상 시스템
- `petitlynn-envelope` — 쁘띠린 사무소 우편봉투 인쇄용 이미지 (버건디+명조체, A5 가로 300dpi)
- `petitlynn-instagram` — 쁘띠린 인스타 매물 게시물 자동 제작 (소재→브랜드카드 PNG+캡션+해시태그 발행패키지, 법정표기 자동, 반자동). 트리거: "인스타 올리자" / "쁘띠린 인스타" / "매물 인스타" / "해시태그 추천"
- `petitlynn-zigbang-image` — 🏠 매물 사진 직방/광고용 실사 보정 (원본무손상·비AI·왜곡금지 → 자연스러운 화이트톤 + 수평 + 4:3/풀프레임 + before/after / 선택 Higgsfield 확장은 허위매물 경고). 트리거: "직방 사진" / "매물 사진 보정" / "사진 화사하게" / "쁘띠린 사진" / "공실 사진 보정"
- `matjib-finder` — 맛집 통합 허브 (일상 즉석 검색 + 여행 끼니 플랜, 카카오 후기 검증 + 확장신호 9종(업력·추세·메뉴·단골·방영·유튜브·인스타·취향)). 트리거: "맛집 찾아줘" / "맛집 추천" / "점심 뭐 먹지" / "여행 맛집" / "식사 플랜" 등. 전신: travel-meal-planner, 2026-08-01 흡수 · 원본은 `~/.claude/_archive/skills-retired/`
- `sauce-lab` — 🍳 소스 레시피 검색/창작/누적 (recipes.md 개인 라이브러리 + AI 창작 하이브리드)

**총 18개** | 트리거 상세 검색은 `skill-manager` 스킬 호출. (2026-07-08: `haemilsia-D0/D2/D3-test` 3종 실험 잔재 아카이브 이동 — `~/.claude/_archive/skills-retired/`)

---

## 일상 스킬 (모드 무관 — 키워드 매칭)

> 💡 트리거 키워드 상세 검색은 `skill-manager` 스킬 호출. 아래는 스킬명 + 핵심 키워드만.

### 1. 문서 생성 (10개)
`docx` / `pdf` / `pptx` / `xlsx` / `pdf-to-knowledge` / `해밀시아기본스킬`(구 land-investment-brochure) / `document-release` / `frontend-slides` / `make-pdf` / `diagram` — Word/PDF/슬라이드/엑셀/브로셔/HTML 발표자료/다이어그램(설명→mermaid·excalidraw·PNG 3종, 2026-08-08 gstack 1.60 신규)

### 2. 문서 읽기 (2개)
`file-reading` / `pdf-reading` — 업로드 파일 / PDF 텍스트·OCR

### 3. 디자인 (16개)
- 기본: `frontend-design` / `design-consultation` / `design-review` / `design-shotgun` / `plan-design-review`
- **메타 워크플로우** ⭐: `design-mood-blend` — 7단계 무드→토큰 (미드저니 + Refero 라이브러리 + 하이브리드 + 목업 비교 + DESIGN_TOKENS 박제 + 에이전트 병렬 코드 + Visual QA). 트리거: "디자인 박제하자" / "DESIGN.md 만들자" / "디자인 리뉴얼" / "무드 블렌드" — 명시 호출만
- 컬러·스타일: `petitlynn-color` / `taste-skill` / `soft-skill` / `minimalist-skill`
- Supanova 랜딩 패키지 (5): `supanova-design-engine` / `supanova-premium-aesthetic` / `supanova-redesign-engine` / `supanova-full-output` / `supanova-report`
- 이미지 리디자인: `japandi-interior-redesign` — 방 사진 → Japandi 스타일 이미지 생성

### 4. 웹 / 배포 (8개)
`landing-page-deploy` / `haemilsia-bot-deploy` / `haemilsia-bot-dev` / `railway-notion-connect` / `ship` / `land-and-deploy` / `setup-deploy` / `canary`

### 5. 자동화 (10개)
`slack-info-briefing-builder` / `terminal-runner` / `browse` / `gstack` / `connect-chrome` / `setup-browser-cookies` / `loop` / `schedule`
- `watch` (claude-video) — 영상 URL·로컬 다운(yt-dlp)→프레임 추출(ffmpeg)→자막 transcript→영상 내용 분석·질의응답. 트리거: "이 영상 분석해줘"/"영상 요약" (bradautomates, skills.sh 설치 / yt-dlp·ffmpeg 필요)
- `insane-search` — 봇 차단(WAF) 사이트 적응형 접근. WebFetch 402/403 시 우회 체인(curl_cffi TLS 위장·모바일 URL·Playwright 실Chrome) 순차 시도. 차단 시에만 발동(일반 검색 X), 첫 호출 시 의존성 자동설치. 트리거: "네이버 블로그 안 열려"/"쿠팡 차단됨"/"유튜브 자막"/"사이트 차단됨" (fivetaku, skills.sh / browse·Chrome 막힐 때 최후수단)

### 6. 품질관리 (9개)
`preflight-check` / `qa` / `qa-only` / `review` / `benchmark` / `investigate` / `cso` / `codex` / `simplify`

### 7. 시스템 / 메타 (13개)
- 스킬 관리: `skill-manager` / `skill-creator` / `system-docs-sync` / `find-skills` — skills.sh 생태계 검색·설치 메타스킬(vercel-labs). 트리거: "X용 스킬 찾아줘"/"~하는 스킬 있어?"/"스킬 설치"
- 파일·화면: `file-organizer` / `screenshot-check`
- 라이프스타일: `sauce-lab` (소스 레시피) / `matjib-finder` (맛집 통합 허브)
- 안전 모드: `freeze` / `unfreeze` / `careful` / `guard`
- 기타: `product-self-knowledge` / `claude-api` / `hook-pack` / `api-key-manager`

### 8. 기획 / 전략 (6개)
`office-hours` / `plan-ceo-review` / `plan-eng-review` / `retro` / `autoplan` / `gstack-upgrade`

### 9. 마케팅 / 광고 (5개)
- `claude-ads` / `ai-marketing-claude` — 광고 감사 / 마케팅 전략·카피·퍼널
- `marketing-strategy-advisor` — 실전 마케팅 전략가 (랜딩/상세/캠페인/카피 진단·브리프)
- `naver-powerlink-advisor` v2.3 ⭐ — 네이버 파워링크 전문 운영 파트너 (5종 캠페인·키워드·소재·확장소재 13종·품질지수·입찰·성과·추적 마스터 + 진단→실행 게이트 + references 14/templates 5 + 자동 스크립트 6종: 기간실적 daily · 그룹/캠페인 오딧 · 발굴 · 등록[dry-run 게이트] · OFF 실행기 — 점검 풀 파이프라인 + **주간 자동 루틴** scheduled-task 월 09:13, 재질 프로필: PU·PE 취급/EVA·EPS·메모리폼 미취급 — 변동 잦음, 스킬 refs/02 표 원본)
- `humanizer` — AI 생성 한국어 텍스트의 AI 흔적 제거 (쉼표 과다·번역투(에 대해/통해)·품사 단조·구조 단조 등 40패턴 S1/S2/S3 교정, KatFishNet 논문 기반). 블로그·카피·상세페이지 초안 자연스럽게. 트리거: "AI 티 빼줘"/"글 자연스럽게"/"humanizer" (daleseo, skills.sh 설치)

### 10. 커뮤니케이션 (9개)
- 슬랙 (7): `slack:find-discussions` / `slack:standup` / `slack:summarize-channel` / `slack:draft-announcement` / `slack:channel-digest` / `slack:slack-messaging` / `slack:slack-search`
- 텔레그램 (2): `telegram:configure` / `telegram:access`

### 11. 투자 리서치 (7개) — 타민더마켓 5종(2026-08-05 설치) + 자체 제작 2종
- `haemilsia-stock-decoder` — 기업 해독기: 티커/기업명 → 한 장 요약 카드 (2분 드릴 · 돈 버는 구조 그림 · 매출 2축 · 업종 KPI · 망하는 시나리오+리스크3 · 재무 5년). 모드 A(웹검색 즉석) / 모드 B(10-K·사업보고서 업로드 시 페이지 출처). 트리거: "[티커] 분석해줘" / "기업 분석" / "요약 카드" / "10-K 분석해줘" / "사업보고서 분석"
  - **v2 개편 (2026-08-13)**: ⑨재무에 **EPS 추가(5지표)** + **이상형 4단 폭포 체크 필수**(매출↑→영업이익 더 빨리↑→EPS 동행→FCF 동행) · ⑪가격표 2줄화(트레일링 + **Forward PER·EV/FCF**) · **공부 카드 스타일 v2(임장기)** — 1주=건물1채 주당 원 단위, 장마다 원리 노트, 예시 3종(일상장사·대표님 부동산·역사실화), 업계 용어 스탬프 노출 필수(자체 축약어만 금지)
- `haemilsia-stock-story` — 스토리 리더: 10-K 여러 해 + 어닝콜 비교 → 사라진/새 문구 · 경영진 톤 변화 · 가이던스 약속 vs 실제를 하나의 스토리로. 트리거: "[티커] 스토리 분석해줘" / "공시 변화 분석" / "가이던스 달성" / "어닝콜 톤 변화"
- `haemilsia-price-decoder` — 가격 판독기: 역DCF로 "현재 주가가 요구하는 성장률" 역산 → 과거 5·10년 실적과 비교. 적정주가·매매신호 제시하지 않음 (조사 방향만). 트리거: "[티커] 지금 사도 되나" / "역DCF" / "밸류에이션 판단" / "이 가격 비싼가"
  - **v2 개편 (2026-08-13)**: **⓪ 가격표 스냅샷** 신설(Forward PER·EV/FCF를 `100÷배수=수익률`로 번역 — "나누고·재고"까지만, 판정은 역DCF) · **공부 카드 스타일 v2(임장기)** 동일 적용 · 모범 예시 = 네이버 판독 v2.1 "226,500원짜리 건물을 보러 간 날"
- `haemilsia-filing-collector` — 공시자료 수집기: SEC EDGAR(10-K 5년·10-Q 8분기·DEF14A) + roic.ai 어닝콜 12분기 URL 수집 → DownThemAll!용 HTML 생성 → 일괄 다운로드 후 "정리해줘"로 `~/Downloads/{티커}_Filings/` 자동 정리. 트리거: "[티커] 공시자료 수집해줘" / "SEC 수집" / "공시 다운로드" / (다운로드 후) "정리해줘"
- `haemilsia-earnings-tracker` — 어닝콜 추적기: Filings 폴더의 기존 어닝콜 스캔 → 누락 분기 roic.ai에서 수집 → 분기별 내러티브·KPI·가이던스·신뢰도 변화 워드 리포트. **Phase 0 (2026-08-07 신설)**: 발표 전 프리뷰 시나리오 — 시장의 믿음에서 역산한 열광 6칸('왕' 1칸 지정)/실망 3칸 채점표 → 발표 후 채점 → 스토리 판정(강화/유지/훼손). 주가 예측 아님, 채점 도구. 트리거: "[티커] 어닝콜 추적" / "어닝콜 업데이트" / "최근 어닝콜 뭐가 달라졌어" / "실적 프리뷰" / "프리뷰 시나리오"
- `stock-discovery-news` — 🗞️ 월간 종목신문 (자체 제작, 2026-08-06): 공부 후보 ≤5개를 기자 4(병렬: 1면 신규통과·2면 급매·3면 버크셔 13F·**4면 미래 대형주**)+편집장+검증 Wave로 발행. 종목 추천 아님(투자헌법 계승) — 발굴·편집만, 심층 분석은 위 4스킬로 연결. 컷라인: 코스닥 500~5,000억·매출3년↑·최근흑자·부채≤200%. 발굴 로그 누적·이월 2회 소멸·보유종목 2면 금지. **4면(rev1, 2026-08-06 신설)**: 성장 렌즈 = 온도계 패널(하모닉드라이브 영업이익률·낙폭 — 로드맵 "찾기 시작할 때" 알림) + 피지컬 AI·로봇·자율주행 신호 기사. 공부 전용 — 후보 5석 밖(render 강제), 미증명 등급 배지 필수(리커전 재발 방지), 컷라인 통과 시 1면 "4면 출신" 졸업 뱃지. args: {ts, today, issue, dryRun, holdings}. 트리거: "신문 발행해줘" / "종목신문" / "이번 달 후보" / "종목 발굴" / "0호 발행" / "OO 공부 끝났어"(로그 전이)
  - **⑬ 재무 6종 이상형 체크 신설 (2026-08-13)**: 전 후보에 `매출→영업이익→EPS→FCF 방향 · Forward PER · Forward EV/FCF` 1줄 필수 + "이상형 n/4단" 뱃지, **정렬 1순위를 이상형 계단 수로 변경**. 확인 불가 항목은 '미확인'(추정 금지), 가격표는 참고 정보이지 컷 기준 아님(종목 추천 금지 헌법 유지)
- `haemilsia-stock-dashboard` — 노션 서재·매매일지 → 관리인의 장부 HTML **지면 3종** 재발행 (전부 고정 URL 유지): **상황판**(허브 一~五) · **매매일지 원장**(접이식·헌법 검인 5종·소급 대기 구획) · **종목 등기부**(종목당 1장 — 해독·가격분석·채점표·판정이력 + 숫자 시각화 4부품: 수치타일·사이클막대·구성띠·스펙트럼 게이지). 도장 클릭→등기부, 일지 행 클릭→원장 앵커. slug 식별자로 URL 4단 폴백. 세션 종료 자동 조건부·논블로킹. **🆕 2026-08-12 — 점검 달력 지난 항목 처리**: 자동 삭제 안 함(안 한 일이 조용히 사라지면 헌법 제5조 2항 근거 소실). 앞으로 할 일이 위, 지난 건 하단 **「⚠ 지연 N건」** 구역으로 분리. 빠지는 경로는 둘뿐 — **수동 일정은 노션 줄 끝에 `✓`**(`done: true`로 전사, 기록은 노션에 잔존) · **종목 점검은 서재「다음 점검일」이동**(서재 파생 행에 `done` 사용 금지). 트리거: "대시보드 갱신해줘" / "상황판 갱신" / "상황판 전체 갱신해줘"(증분 무시·강제 전량) / "일지 갱신해줘"(원장만) / "상황판 보여줘"(재발행 없이 전송)
> 권장 순서: **종목신문(발굴)** → 수집기 → 해독기 → 스토리 리더 → 가격 판독기 → (분기마다) 어닝콜 추적기. 두 스킬이 참조하는 `earnings-update-engine-korean` · `10k-evolution-report`는 제작자 미배포분 — 현재 미설치.

---

*Haemilsia AI operations | 2026.08.08 | skill-guide v3.6 — GSD 1.34.2→1.42.3·gstack 1.57.4→1.60.2·superpowers 7/28분·claude-ads v2.0.1 업그레이드 반영 (skillOverrides 65개 재정렬 · GSD 통합 명령 안내 · make-pdf/diagram 등록)*
