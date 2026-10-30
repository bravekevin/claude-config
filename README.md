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

## 새 Mac 준비 (2026-10-31)
프로젝트와 무관한 공통 준비다. 프로젝트별 준비(키 이름·민감 폴더·DB 도구)는 각 프로젝트 저장소의 `runbooks/` 를 따른다 — 이 공개 저장소에는 적지 않는다.

1. **도구**: Claude 데스크톱 앱(로그인 후 Remote Control 켜기), Homebrew, git, `gh`(작업 계정을 활성으로), Node 22(nvm), Docker Desktop, Chrome + Claude in Chrome.
2. **전역 지침**: 이 저장소를 받아 `~/.claude/CLAUDE.md` 를 import 두 줄로 둔다(위 '연결'). 기존 파일이 있으면 먼저 백업한다.
   ```bash
   git clone https://github.com/bravekevin/claude-config.git ~/.claude-config
   ```
3. **지침 자동 갱신 훅**: 세션 시작마다 이 저장소를 당겨 온다.
   ```bash
   S=~/.claude/settings.json; [ -f "$S" ] || echo '{}' > "$S"
   C='git -C "$HOME/.claude-config" pull -q --ff-only 2>/dev/null || true'
   jq --arg c "$C" '.hooks.SessionStart = (((.hooks.SessionStart // []) | map(select(.hooks[0].command != $c))) + [{hooks: [{type: "command", command: $c, timeout: 30, statusMessage: "개인 전역 지침 갱신 중"}]}])' "$S" > "$S.tmp" && mv "$S.tmp" "$S"
   ```
4. **`tok`**: 옛 Mac 의 `~/Desktop/Dev/bin/tok`·`TOKEN_POLICY.md`·`TOKEN_INVENTORY.md` 를 직접 전달로 옮기고 `~/Desktop/Dev/bin` 을 PATH 에 넣는다. Keychain 값은 옮겨지지 않는다 — 각 프로젝트 runbook 의 이름 목록대로 `tok set` 으로 다시 넣는다. 인벤토리에는 키 이름이 있으니 이 저장소에 올리지 않는다.
5. **프로젝트 메모리**: 옛 Mac 의 `~/.claude/projects/<경로>/memory/` 를 옮긴다. 폴더 이름은 프로젝트 절대경로에서 만들어지므로 같은 경로로 clone 한다(사용자 이름이 다르면 폴더 이름을 맞춘다).
6. **확인**: 새 세션에서 이 저장소의 지침(한글·caveman 등)이 적용되는지 본다. `/context` 의 Memory files 에 `~/.claude-config/CLAUDE.md`·`mac.md` 가 보이면 된다.
