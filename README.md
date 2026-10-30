# claude-config — 개인 Claude Code 전역 지침 정본

Mac 여러 대와 클라우드 세션이 같은 지침을 쓰기 위한 저장소.

**공개 저장소다(2026-10-30 전환 — 클라우드 세션이 인증 없이 받게).** 비밀값·이메일·회사 내부 정보는 넣지 않는다(이름·절차만).

| 파일 | 쓰는 곳 |
|---|---|
| `CLAUDE.md` | 공통 — 한글·caveman·karpathy·커밋·테스트·프로젝트 지침 우선순위 |
| `mac.md` | Mac 만 — Keychain(`tok`) 토큰 정책 |
| `cloud.md` | 클라우드만 — 네트워크 시크릿 전제의 비밀값 규칙 |

## 연결
- Mac: `git clone https://github.com/bravekevin/claude-config.git ~/.claude-config` 뒤 `~/.claude/CLAUDE.md` 를 아래 두 줄로 둔다. 사용자 수준 SessionStart 훅이 `git -C ~/.claude-config pull --ff-only` 로 갱신한다.
  ```
  @~/.claude-config/CLAUDE.md
  @~/.claude-config/mac.md
  ```
- 클라우드: 환경 설정 스크립트가 clone 하고 `~/.claude/CLAUDE.md` 를 `@~/.claude-config/CLAUDE.md` + `@~/.claude-config/cloud.md` 로 쓴다.

지침 수정은 이 저장소 커밋으로만 한다. `~/.claude/CLAUDE.md` 를 직접 고치지 않는다.
