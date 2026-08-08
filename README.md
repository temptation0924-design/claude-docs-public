# claude-docs-public

Haemilsia AI operations 지침의 **공개 미러**입니다. Claude.ai가 raw URL로 지침을 읽는 용도로만 존재합니다.

- **원본**: 로컬 `~/.claude` (원본 리포 `claude-system-docs`는 **private**)
- **빌드·동기화**: `bash ~/.claude/code/build-integrated_v1.sh --push`
- **담는 것**: 지침 md만 — `INTEGRATED.md`, `CLAUDE-core_v1.md`, 개별 문서 8종, `on-demand/mode*.md`

## 여기에 두지 않는 것

핸드오프, 세션 기록, 메모리, 업무 데이터, 임차인·거래 정보, 토큰·API 키. 2026-08-09 리포 분리의 이유가 이것입니다.

빌드 스크립트에 가드가 있어, 예상 외 파일이 섞이면 push가 중단됩니다.
