# 설계 전용 하네스 (Claude 설계 + Codex 구현)

이 프로젝트는 Claude가 설계만, 구현은 Codex CLI가 담당하는 분업 구조다.
Claude는 코드를 직접 짜지 않는다. 인터뷰→매뉴얼→AGENTS.md export까지만 한다.

## 코드맵 (codemap.md)

`docs/codemap.md`는 각 파일이 무엇을 export/import하는지 자동 생성한 지도다.
Codex가 짠 코드를 Claude 세션에서 파악할 때 유용하다.

- **세션 시작 시 자동 로드**(SessionStart 훅) — Codex가 만든 코드 구조를 빠르게 파악.
- **작업 종료 시 자동 갱신**(Stop 훅).
- 수동 갱신: `python3 .claude/hooks/codemap.py`
- 직접 수정 금지(자동 생성물).

## 핵심 산출물
- `docs/domains/<domain>/interview.json` — 인터뷰 결정 기록
- `docs/domains/<domain>/manual.md` — 도메인 매뉴얼(규칙)
- 각 code_path의 `AGENTS.override.md` — Codex용 규칙 (매뉴얼에서 export)

## 코딩 규칙은 Codex에게 전달된다
이 하네스에선 Claude가 코드를 짜지 않으므로, 전역 코딩 규칙(단순성·외과적 변경·네이밍·
보안 등)과 스택별 규칙(프론트/백엔드)은 Claude가 아니라 **Codex가 따라야** 한다.
따라서 이 규칙들은 CLAUDE.md가 아니라 매뉴얼 export 시 `AGENTS.override.md`에 포함되어
Codex에게 전달된다. domain-manual 스킬의 export 절을 참고.

## 슬래시 커맨드

이 하네스는 설계 전용이라 설계 단계 커맨드만 둔다(구현 커맨드는 Codex가 담당).

- `/grill` — 코딩 전 요구사항을 질문 공세로 명확히 한다.
- `/plan-start` — 기획 시작. ui-planner로 요구사항 정리.
- `/architect` — 도메인 엔티티·타입 스캐폴딩 생성(인터페이스 수준, 구현은 Codex).
- `/commit` — 변경사항 분석 후 conventional commit 생성.
