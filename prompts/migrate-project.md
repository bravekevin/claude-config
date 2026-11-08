# 지시문 — 프로젝트 하나를 GitHub 중심 환경으로 옮기기

사용자는 세션에 이렇게 붙여 넣는다.
> ~/.claude-config/prompts/migrate-project.md 대로 <프로젝트 폴더 경로> 를 옮겨줘.

`~/.claude-config` 가 없으면 먼저 `prompts/new-mac-setup.md` 를 한다. 저장소가 있는 그 컴퓨터(옛 폴더가 있는 컴퓨터)에서 실행한다.

---

## Claude가 할 일

### 0. 지켜야 할 것
- 비밀 값을 읽거나 출력하지 않는다. env 파일은 `cp` 로 복사만 한다.
- 다른 세션이 쓰는 폴더·worktree를 옮기거나 지우지 않는다. 옛 폴더 정리는 `prompts/retire-old-folders.md` 로 따로 한다.
- 팀·조직 저장소의 공용 브랜치(main, dev, develop 등)에 바로 push하지 않는다. 그런 브랜치의 로컬 커밋은 `git-safety` 가 `backup/*` 로 올린다. 공용 브랜치에 올려야 하면 사용자에게 묻는다.
- GitHub 저장소를 새로 만드는 일, 원격에 쓰는 일은 계획을 보여 주고 사용자 승인 뒤에 한다. 사용자가 "모두 진행"이라고 했으면 그 승인으로 본다.

### 1. 점검
```bash
git-safety audit <옛 폴더>
```
결과를 요약한다. git 저장소가 아니면 2-a 로.

### 2-a. 원격이 없으면 (또는 git 저장소가 아니면)
- 비밀 파일이 커밋될 위험부터 본다: `.gitignore` 에 `.env*`, 키 파일, 빌드 폴더가 빠져 있으면 먼저 보강한다. `git status` 로 올라갈 파일 목록을 보여 준다(파일 이름만).
- git 저장소가 아니면 `git init` → 첫 커밋.
- 사용자에게 저장소 이름·계정(개인/조직)·비공개 여부를 확인하고 `gh repo create <계정>/<이름> --private --source <옛 폴더> --remote origin` → `git push -u origin <기본 브랜치>`.

### 2-b. 백업
```bash
git-safety backup <옛 폴더>          # 계획 확인
git-safety backup --yes <옛 폴더>    # 실행
git-safety audit <옛 폴더>           # [위험] 0 확인
```
- 저장소 계정이 `gh` 활성 계정과 다르면(`gh auth switch`) 사용자에게 알린다.
- 내 브랜치 접두사가 `claude/` 가 아니면 `--own <접두사>` 를 쓴다.

### 3. 새 위치
```bash
cd ~/Dev && gh repo clone <owner/repo> <폴더이름>
```
- 폴더 이름은 옛 폴더 이름과 같게 한다(Claude 메모리 연결 때문).
- 옛 폴더의 env 파일을 같은 상대 경로로 `cp -p` 한다. 찾기: `find <옛 폴더> -maxdepth 4 \( -name '.env*' -o -path '*/.vercel/project.json' \) -not -path '*/node_modules/*' -not -name '*.example'`. 이름만 보고, 내용은 열지 않는다.
- 의존성 설치 → 테스트 실행 → 결과 숫자 보고.
- `python3 ~/.claude-private/bin/link-memory` 로 메모리 링크 확인.
- `git-safety audit ~/Dev/<폴더이름>` → "안전".

### 4. 기록
- `~/.claude-private/PROJECTS.md` 의 그 프로젝트 줄을 "적용 완료"로 옮기고 날짜·남은 일을 적는다 → `bash ~/.claude-private/bin/sync push`.
- 프로젝트에 `reports/` 같은 기록 규칙이 있으면 거기에도 남긴다.

### 5. 보고
- 올린 브랜치 목록, 새 위치, 테스트 결과, 남은 일(env 값, 옛 폴더 정리).
