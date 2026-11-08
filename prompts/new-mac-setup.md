# 지시문 — 새 Mac(또는 아직 준비 안 된 Mac)을 GitHub 중심 개발 환경으로 준비

이 파일은 Claude 세션에 주는 지시문이다. 사용자는 세션에 이렇게만 붙여 넣는다.
> https://github.com/bravekevin/claude-config 의 prompts/new-mac-setup.md 대로 이 Mac을 준비해. 프로젝트: <owner/repo> [<owner/repo> ...]

---

## Claude가 할 일 (순서대로, 각 단계 결과를 한 줄씩 보고)

### 0. 지켜야 할 것
- 비밀 값(env 파일 내용, 토큰, 비밀번호)을 읽거나 출력하지 않는다. 비밀번호 입력·로그인은 사용자가 직접 한다.
- `~/Desktop`, `~/Documents` 아래에 clone하지 않는다(iCloud 동기화 폴더). 작업 위치는 `~/Dev/<폴더>`.
- 기존 파일을 덮어쓸 때는 백업이 먼저다(`dev-bootstrap`이 알아서 한다).
- main·공용 브랜치 push, force push 금지.

### 1. 사전 확인
- `git`, `gh`, `node`, `pnpm`(또는 프로젝트가 쓰는 패키지 관리자), `python3` 가 있는지 본다. 없으면 설치 명령을 사용자에게 안내한다(Homebrew 설치는 사용자가 한다).
- `gh auth status` — 로그인 안 돼 있으면 멈추고 사용자에게 `gh auth login` 을 터미널에서 직접 해 달라고 한다. 끝났다고 하면 이어 간다.

### 2. 전역 지침·도구
```bash
[ -d ~/.claude-config/.git ] && git -C ~/.claude-config pull -q --ff-only || git clone -q https://github.com/bravekevin/claude-config.git ~/.claude-config
~/.claude-config/bin/dev-bootstrap
```
`dev-bootstrap` 이 하는 일: `~/Dev` 생성, 비공개 저장소 `claude-private` clone, Claude 메모리 링크, 동기화 hook, `~/.claude/CLAUDE.md` 전역 지침 연결, PATH.
실패하면 오류 원문을 그대로 보여 주고 원인을 확인한다(추측해서 고치지 않는다).

### 3. 프로젝트 받기 (사용자가 준 저장소마다)
```bash
cd ~/Dev && gh repo clone <owner/repo> <폴더이름>
```
- 폴더 이름은 다른 컴퓨터와 같게 한다(보통 저장소 이름, 사용자가 정한 이름이 있으면 그것). 같은 이름이어야 Claude 메모리가 이어진다.
- 이미 있으면 `git -C ~/Dev/<폴더> pull --ff-only`.
- 의존성 설치(`pnpm install` 등). 경고·오류는 원문을 보고 판단한다. 패키지가 설치됐는지는 테스트로 확인한다.

### 4. env 파일
- 저장소 `.gitignore` 에 걸린 env 파일(`.env.local` 등)은 GitHub에 없다. 내용을 읽지 않는다.
- Vercel 프로젝트가 연결된 앱이면 사용자에게 "`vercel env pull` 로 받을까요? (Vercel 개발 환경 값이 들어갑니다)" 하고 묻고, 승인 뒤 실행한다. 아니면 사용자가 금고에서 직접 넣도록 파일 경로만 알려 준다.

### 5. 확인
- 프로젝트 테스트를 실행한다(`pnpm test` 등). 결과 숫자를 보고한다.
- `git-safety audit ~/Dev/<폴더>` → "안전" 확인.
- `git -C ~/.claude-private log -1 --oneline` 으로 메모리 저장소가 받아졌는지 확인.

### 6. 보고
- 단계별 결과 표, 실패·건너뛴 것과 이유.
- 사용자가 할 일: env 값, `tok set <키이름>`(키 이름은 `~/.claude-private/tokens/TOKEN_INVENTORY.md`), Claude 새 세션은 "이 컴퓨터" → `~/Dev/<폴더>` 로 열기.
