# claude codex design-harness — 설계 전용 Claude 하네스 (Codex 구현 연동)

**Claude(Opus)는 설계만, 구현은 Codex CLI가** 하는 분업 워크플로우용 하네스입니다.
Claude는 인터뷰로 요구사항을 모으고, 매뉴얼로 규칙을 문서화하고, 그 규칙을
Codex가 읽는 `AGENTS.override.md`로 내보내는 데까지만 책임집니다.
코딩·테스트·디버깅은 Codex CLI가 담당합니다.

## 책임 경계

| 단계 | 담당 | 산출물 |
|------|------|--------|
| 도메인 발견·인터뷰·모순 검출 | Claude | `docs/domains/_discovery/`, `<domain>/interview.json` |
| 매뉴얼·도메인 지도 문서화 | Claude | `manual.md`, `docs/domains/README.md` |
| Codex 핸드오프 (규칙 export) | Claude | 각 code_path의 `AGENTS.override.md` |
| **코딩·테스트(TDD)·디버깅** | **Codex CLI** | `src/...` 실제 코드 |
| 사후 검증 (코드↔매뉴얼) | Claude | drift_guard 경고, domain-refactor 리포트 |

Claude는 코드를 직접 짜지 않습니다. 구현이 필요하면 매뉴얼을 export하고 Codex로 넘깁니다.

## 구성

### skills/ (설계용만)
- `domain-interview` — 도메인 발견 + 인터뷰 + 모순 검출
- `domain-manual` — 매뉴얼/도메인 지도 생성·갱신 + **Codex용 AGENTS.override.md export**
- `domain-refactor` — 매뉴얼 규칙 대비 코드 진단 (Codex가 짠 코드 점검에 사용, report 모드)
- `grill` — 단일 기능 빠른 정렬
- `architecture` — 설계 품질 (deep module 등)
- `karpathy-guidelines` — 일반 개발 가이드라인

> 구현용 스킬(tdd, diagnose 등)은 의도적으로 제외했습니다. Codex가 담당합니다.

### hooks/ (설계+감시용만)
- `drift_guard.py` — 코드↔매뉴얼 어긋남 감시 (Codex 코드를 Claude 세션에서 열 때 발동)
- `security_gate.py` — Bash 명령 안전 검사 (범용)

> 구현 리뷰 훅(code_reviewer 등)은 제외했습니다.

### agents/, commands/ (설계용만)
architect, ui-planner / grill, plan-start, architect, ui-design, qa-boundary, lessons, commit

## 워크플로우

1. **설계 (이 하네스, Claude Code)**
   - "도메인 설계 시작" → domain-interview가 Phase 0(발견)부터 인터뷰
   - 인터뷰 complete → domain-manual로 매뉴얼 생성
   - 매뉴얼 complete → domain-manual의 export 절로 `AGENTS.override.md`를 code_paths에 배치
2. **구현 (Codex CLI, 별도 실행)**
   - 코드 디렉터리에서 Codex 실행 → AGENTS.override.md 읽고 규칙대로 구현·테스트
3. **검증 (다시 이 하네스)**
   - drift_guard가 어긋남 감시, domain-refactor로 위반 진단

자세한 핸드오프 방법은 `.claude/skills/domain-manual/references/codex-handoff-guide.md` 참고.

## 한계 (정직한 현황)
- Codex CLI 훅은 현재 apply_patch(파일편집)에 발동하지 않아, Codex 쪽 실시간 차단은 불가.
  방어선은 "Codex는 AGENTS.md로 예방 + Claude가 사후 검출(drift_guard·domain-refactor)".
- 매뉴얼↔AGENTS.override.md 동기화는 수동(매뉴얼 갱신 시 재export 필요).
- 설계/구현 경계 중 일부(에러처리·디자인 등)는 운영하며 조정 필요.
