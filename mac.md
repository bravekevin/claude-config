# Mac 전용 전역 지침 — Keychain(tok) 토큰 정책. CLAUDE.md 와 함께 import 한다

## 🔐 토큰/시크릿 취급 (필수 준수)

**정책:** `~/Desktop/Dev/TOKEN_POLICY.md`
**인벤토리:** `~/Desktop/Dev/TOKEN_INVENTORY.md`
**CLI:** `~/Desktop/Dev/bin/tok`

### 한 줄 원칙
장기보관 = Keychain(사용자 확인 필수). 실행 시 주입 = 1Password/pass. **env 는 저장소가 아니라 일회성 전달 수단**.

### 조회 기본은 `run`
- ✅ `tok run <key> <ENV_NAME> -- <command>` — 하위 프로세스에만 주입
- ✅ `tok env <ENV_NAME> <key> -- <command>` — 동의어
- ⚠️ `tok read <key> --reason "<text>"` — stdout 평문 노출. `--reason` 필수이며, 이유는 사용자에게 **먼저 확인**한 뒤 전달

### Tier 판정 먼저 (토큰 받을 때)
세션에서 토큰을 받으면 **저장 전에 Tier 부터 판정**:
- **T1 (개인 장기보관, 로컬 단일 사용자, 고권한 PAT 등)** → `tok set` 제안
- **T2 (팀/크로스플랫폼 공유)** → `op` / `pass` 사용 안내, **`tok set` 금지**
- **T3 (CI/서버 전용)** → CI Secret Manager 에서만 관리하도록 안내, **로컬 저장 금지**
판정이 애매하면 사용자에게 확인.

### 반드시 할 것
- 토큰 필요 작업은 **`tok run` 형태로 제안**
- 신규 토큰 발급 요청 시 **단수명/OIDC/더 낮은 스코프 대안 먼저 제시**
- 만료 없는 사용자 OAuth access token 발급은 **거부**하고 refresh flow 권고
- T1 으로 판정된 토큰만 `tok set` 제안. 저장 후 `TOKEN_INVENTORY.md` 엔트리 추가(메타데이터만) 제안
- 평문은 응답 본문/메모리에서 즉시 제거
- 토큰 사용/조회 시 **사용자에게 명시적으로 고지**

### 절대 하지 말 것
- `cat ~/.env.secrets`, `cat **/.env.local`, `cat ~/.ssh/*` 등 **시크릿 파일 무단 조회 금지**
- 토큰 값을 **응답 본문·memory/\*.md·코드 주석·커밋 메시지·PR 본문에 평문 기재 금지**
- `tok read` 결과를 **파일로 리다이렉트 / 다른 변수에 장기 저장 금지**
- 토큰 인자를 포함한 명령을 `.claude/settings.json` **allow 리스트에 추가 금지**
- `export TOKEN=...` 식으로 **쉘 히스토리에 남기는 패턴 제안 금지**
- `TOKEN_POLICY.md` / `TOKEN_INVENTORY.md` 에 **값 기입 금지** (이름/메타데이터만)

### 메모리 저장 규칙
메모리 `.md` 파일에 토큰 값 저장 금지. 포인터만 허용:
```
✓ OK:  "Notion API 토큰은 `tok run gentlefarmers.notion.api_token NOTION_TOKEN -- ...` 로 사용"
✗ 금지: "Notion Token: ntn_abc123..."
```

### Tier 참조 (상세는 TOKEN_POLICY.md §2)
- **T1** 개인 장기보관: `login.keychain-db` + `-T ""` ACL (`tok`). ⚠️ `-T ""`는 "생성 프로세스 자동 신뢰 제거" 수준이며 per-read 생체 강제는 아님
- **T2** 팀/크로스플랫폼: 1Password CLI (`op run`) 우선, 대안 `pass`. per-operation 생체 인증이 필요한 경우도 T2 격상 고려
- **T3** CI/서버: CI Secret Manager 전용, 로컬 장기보관 금지
