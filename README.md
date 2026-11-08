# claude-config — 개인 Claude Code 전역 지침 정본

Mac 여러 대와 클라우드 세션이 같은 지침을 쓰기 위한 저장소.

**공개 저장소다(2026-10-30 전환 — 클라우드 세션이 인증 없이 받게).** 비밀값·이메일·회사 내부 정보는 넣지 않는다(이름·절차만).

| 파일 | 쓰는 곳 |
|---|---|
| `CLAUDE.md` | 공통 — 한글·caveman·karpathy·커밋·테스트·프로젝트 지침 우선순위 |
| `mac.md` | Mac 만 — Keychain(`tok`) 토큰 정책 |
| `cloud.md` | 클라우드만 — 네트워크 시크릿 전제의 비밀값 규칙 |
| `dev-sync.md` | GitHub 중심 개발 환경 — 분실 대비·여러 컴퓨터 동기화 안내 |
| `bin/git-safety` | 저장소 점검(`audit`)·GitHub 백업(`backup`) |
| `prompts/*.md` | Claude 지시문: 새 Mac 준비·프로젝트 옮기기·옛 폴더 정리 |
| `bin/dev-bootstrap` | 새 Mac 준비: 비공개 저장소 clone·메모리 링크·hook·PATH |

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
3. **자동 준비(2026-11-08)**: 아래 한 줄이 지침 갱신 hook, 비공개 저장소(`<계정>/claude-private`: 메모리·`tok` 정책 사본·분실 대응표) clone, 메모리 링크, 메모리 동기화 hook, PATH 를 맞춘다. 여러 번 실행해도 안전하다.
   ```bash
   ~/.claude-config/bin/dev-bootstrap
   ```
4. **`tok`**: 스크립트·정책 사본은 비공개 저장소 `tokens/` 에 있다. Keychain 값은 옮겨지지 않는다 — 인벤토리의 이름 목록대로 `tok set` 으로 다시 넣는다(키 이름이 있으니 이 공개 저장소에는 올리지 않는다).
5. **프로젝트**: `~/Dev/<프로젝트>` 에 clone 한다(iCloud 밖, 모든 컴퓨터 같은 경로). 자세한 순서는 `dev-sync.md`.
6. **확인**: 새 세션에서 이 저장소의 지침(한글·caveman 등)이 적용되는지 본다. `/context` 의 Memory files 에 `~/.claude-config/CLAUDE.md`·`mac.md` 가 보이면 된다.
