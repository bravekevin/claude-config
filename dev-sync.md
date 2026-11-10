# GitHub 중심 개발 환경 — 분실 대비·여러 컴퓨터 동기화

목표: 어느 컴퓨터가 꺼지거나 없어져도 작업이 사라지지 않고, 어느 컴퓨터(Mac 여러 대·클라우드 세션)에서든 이어서 작업한다.

## 원칙
1. **코드 동기화는 GitHub만.** iCloud·Dropbox 폴더에 git 저장소를 두지 않는다. 파일 단위 동기화가 `.git`을 덮어쓰고 조용히 "파일 2" 충돌 사본을 만든다.
2. **작업 위치는 `~/Dev/<프로젝트>`.** `~/Desktop`, `~/Documents`는 iCloud "데스크탑 및 문서 폴더"가 켜져 있으면 동기화되므로 쓰지 않는다. 모든 컴퓨터에서 같은 경로를 쓰면 Claude 메모리도 같은 경로키로 이어진다.
3. **작업이 끝나면 push.** 다른 컴퓨터에 앉으면 먼저 pull. 한 브랜치는 한 곳에서만 작업한다.
4. **worktree는 영구 경로에.** `/tmp`는 재부팅하면 비워진다.
5. **비밀 값은 git에도 iCloud에도 두지 않는다.** env 파일은 `vercel env pull` 또는 금고(1Password 등)에서 받고, 장기 키는 각 컴퓨터 키체인(`tok`)에 다시 넣는다. 재발급할 수 없는 키(지갑 개인키, 백업 복호화 키)는 반드시 금고에 사본을 둔다.

## 구성
| 저장소 | 공개 | 내용 |
|---|---|---|
| `claude-config` (`~/.claude-config`) | 공개 | 전역 지침, `bin/git-safety`, `bin/dev-bootstrap`, 이 안내서 |
| `claude-private` (`~/.claude-private`) | **비공개** | Claude 메모리, 토큰 정책 사본(값 없음), 분실 대응표, 프로젝트 상태표 |
| 각 프로젝트 저장소 | 프로젝트별 | 코드 |

Claude hook(`~/.claude/settings.json`)
- SessionStart: `claude-config` pull, `claude-private` pull + 메모리 링크
- Stop: `claude-private` push(바뀐 메모리만, 비밀 값처럼 보이면 멈춤)

## 도구
### `git-safety audit [경로...]` — 읽기 전용 점검
iCloud 안 위치, `.git` 충돌 사본, 원격에 없는 커밋, 커밋 안 된 변경(스냅샷 백업 여부 포함), 임시 폴더 worktree, stash를 보여 준다.

### `git-safety backup [--yes] [--own 접두사] [경로]` — GitHub에 없는 작업 올리기
`--yes` 없이는 계획만 보여 준다.
| 대상 | 올라가는 곳 |
|---|---|
| upstream 있고 fast-forward 가능 | 원래 upstream |
| 내 접두사(기본 `claude/`) 브랜치, upstream 없음 | 같은 이름(새 원격 브랜치) |
| 다른 사람·도구 브랜치, 갈라진 브랜치 | `backup/<브랜치>` (원래 이름은 건드리지 않음) |
| main/master 로컬 커밋 | `backup/main` (main 자체에는 push 안 함) |
| stash | `backup/stash/<번호>-<SHA>` |
| 커밋 안 된 변경 | `backup/wip/<날짜-시각>/<폴더>` 스냅샷 (작업 폴더·index는 안 건드림, 빌드 폴더·95MB 초과 파일 제외) |

force push는 하지 않는다. 다시 실행하면 이미 올라간 것은 건너뛴다.

## Claude에게 시키기 (지시문)
| 하고 싶은 것 | 세션에 붙여 넣을 말 |
|---|---|
| 새 Mac 준비 | `https://github.com/bravekevin/claude-config 의 prompts/new-mac-setup.md 대로 이 Mac을 준비해. 프로젝트: <owner/repo>` |
| 프로젝트 하나 옮기기 | `~/.claude-config/prompts/migrate-project.md 대로 <옛 폴더> 를 옮겨줘.` |
| 옛 폴더 정리 | `~/.claude-config/prompts/retire-old-folders.md 대로 <옛 폴더> 를 정리해줘.` |

## 프로젝트 하나 옮기기 (순서)
1. `git-safety audit ~/Desktop/Dev/<프로젝트>` — 상태 확인
2. 원격이 없으면 만든다: `gh repo create <계정>/<이름> --private --source ~/Desktop/Dev/<프로젝트> --remote origin` 후 `git push -u origin main`
3. `git-safety backup ~/Desktop/Dev/<프로젝트>` 로 계획 확인 → `--yes` 로 실행 → 다시 audit 해서 [위험] 0 확인
4. `cd ~/Dev && gh repo clone <계정>/<저장소> <프로젝트>`
5. 의존성 설치, env 복사(옛 폴더에서 `cp`) 또는 `vercel env pull`, 테스트 실행
6. Claude 새 세션은 `~/Dev/<프로젝트>` 로 연다
7. 옛 폴더에서 돌던 세션이 다 끝나면 옛 폴더를 정리한다(`backup/wip` 스냅샷이 있으니 되돌릴 수 있음)

팀·조직 저장소는 main·dev 같은 공용 브랜치에 바로 push하지 말고 팀 규칙을 따른다. `backup/*`로 올리는 것은 안전하다.

## 새 컴퓨터 (처음 한 번)
가장 빠른 방법: Claude 앱 설치·로그인 → 터미널에 한 줄. 비밀번호 입력과 GitHub·Vercel 브라우저 로그인만 직접 한다.
```bash
curl -fsSL https://raw.githubusercontent.com/bravekevin/claude-config/main/bin/mac-setup | bash
```
프로젝트를 물으면 `계정/저장소:폴더` 로 입력한다(예: `bravekevin/zellytalk:ZellyTalk`). 아래는 같은 일을 손으로 하는 순서다.

```bash
# 1) 도구: Homebrew, git, gh, Node, Claude 데스크톱 앱
gh auth login
# 2) 전역 지침과 도구
git clone https://github.com/bravekevin/claude-config.git ~/.claude-config
# 3) ~/.claude/CLAUDE.md 를 import 두 줄로(README '연결' 참고)
# 4) 나머지 자동 준비: 비공개 저장소 clone, 메모리 링크, hook, PATH
~/.claude-config/bin/dev-bootstrap
# 5) 프로젝트 받기
cd ~/Dev && gh repo clone <계정>/<저장소> <폴더이름>
```
Claude 데스크톱 앱: 새 세션 → "이 컴퓨터" → `~/Dev/<폴더이름>` 선택. 폴더 안 `.git`이 GitHub와 연결돼 있으므로 따로 저장소를 고를 필요가 없다.

### 비공개 저장소 처음 만들기 (계정당 한 번)
`~/.claude-private`에 `bin/link-memory`, `bin/sync`를 두고 `gh repo create <계정>/claude-private --private --source ~/.claude-private --push`. 그 뒤로는 `dev-bootstrap`이 clone한다.

## 비밀 값(env) — 암호화 보관 (2026-11-10 사용자 결정)
- 각 프로젝트의 `.env*`·`.dev.vars`·`.vercel/project.json` 은 비공개 저장소 `claude-private/env/<프로젝트>.tar.age` 에 **age 암호문**으로만 보관한다. 도구는 `claude-private/bin/env-vault`.
- 암호화는 공개키(`env/recipient.txt`)만으로 된다 → Claude 세션이 값을 보지 않고 저장할 수 있다. 복호화 열쇠는 사용자가 정한 암호로 잠근 `env/identity.age`. 각 Mac 의 풀린 열쇠는 `~/.config/env-vault/identity`(600).
- **새 비밀이 생기면**: 프로젝트의 `.env.local` 등에 넣기만 한다. Claude 응답이 끝날 때 도는 Stop hook(`sync push`)이 바뀐 env 만 골라 자동 암호화·push 한다. 다른 Mac 은 `env-vault restore <프로젝트>`.
- 새 Mac: `mac-setup` 9단계가 열쇠 암호를 한 번 묻고 모든 프로젝트 env 를 복원한다.
- 처음 한 번(계정당): `env-vault init` — 열쇠를 만들고 암호를 두 번 입력. 그 암호는 Apple 암호 앱 등에 따로 적어 둔다(잃으면 보관본을 못 연다).
- 장기 개인 키(`tok`, 키체인)는 이 방식과 별개로 컴퓨터마다 `tok set`.

## 컴퓨터를 잃어버렸을 때
`~/.claude-private/LOST_DEVICE.md` (비공개 저장소)를 따른다. 요지: 나의 찾기로 잠금 → GitHub·Vercel·Supabase·Claude 세션과 CLI 토큰 폐기 → 키체인 키 재발급 → 새 컴퓨터는 위 순서.
