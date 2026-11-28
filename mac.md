# Mac 전용 전역 지침 — Keychain(tok) 토큰 정책. CLAUDE.md 와 함께 import 한다

## 🔐 토큰/시크릿 취급 (필수 준수)

정책 `~/Desktop/Dev/TOKEN_POLICY.md`(Tier 상세 §2) · 인벤토리 `~/Desktop/Dev/TOKEN_INVENTORY.md` · CLI `~/Desktop/Dev/bin/tok`. 장기보관은 Keychain, 실행 시 주입은 1Password/pass. **env 는 저장소가 아니라 일회성 전달 수단**이다.

- **조회는 `tok run <key> <ENV_NAME> -- <command>`**(동의어 `tok env`) — 하위 프로세스에만 주입한다. `tok read <key> --reason "<이유>"` 는 평문이 나오므로 이유를 사용자에게 먼저 확인하고, 결과를 파일·변수에 남기지 않는다. 토큰을 쓰거나 조회할 때는 사용자에게 알린다.
- **토큰을 받으면 저장 전에 Tier 판정**(애매하면 묻는다). T1 개인 장기보관 → `tok set` 제안 후 인벤토리에 메타데이터만 추가 제안. T2 팀·크로스플랫폼 → `op`/`pass`, `tok set` 금지. T3 CI/서버 → CI Secret Manager 만, 로컬 저장 금지.
- **새 토큰 발급**은 단수명·OIDC·낮은 스코프 대안을 먼저 제시한다. 만료 없는 사용자 OAuth access token 은 거부하고 refresh flow 를 권한다.
- **금지**: 시크릿 파일 무단 조회(`~/.env.secrets`, `**/.env.local`, `~/.ssh/*`), 토큰 값을 답변·메모리·코드 주석·커밋·PR 에 평문으로 적기(메모리에는 `tok run <key> ...` 같은 포인터만), 토큰 인자가 든 명령을 `settings.json` allow 에 추가, `export TOKEN=...` 처럼 히스토리에 남는 패턴, 정책·인벤토리 파일에 값 기입. 평문이 보이면 즉시 지운다.
- **키 보관본 `tok-vault`**(2026-11-23): T1 키 전체를 age 로 암호화한 보관본 하나만 예외로 허용(비공개 저장소 `tokens/TOKEN_POLICY.md` §8-1). FileVault 가 켜진 Mac 에서만 `tok-vault save`/`restore`, 평문 파일·명령 인자로 값을 옮기지 않는다. 분실 시 폐기·재발급 원칙은 그대로.

## ▲ Vercel CLI는 `vercel-as` 로 (2026-11-11 사용자 결정)

전역 로그인은 계정 하나뿐이라 한 세션의 `vercel login` 이 다른 프로젝트 배포를 깬다.

- `vercel` 대신 `vercel-as` 로 실행한다(인자는 그대로). 위쪽 폴더의 `.vercel-account`(계정 이름)로 `~/.vercel-accounts/<이름>` 설정을 쓴다. 파일이 없으면 기본 `vercel`.
- 전역 `vercel login`·`logout` 금지. 새 계정은 프로젝트에 `.vercel-account` 를 두고 사용자가 Mac 마다 `vercel-as login` 을 한 번 한다.
- `whoami` 가 예상 계정이 아니면 배포하지 않고 알린다.

## 🖥️ 화면 제어(computer-use) 승인은 모바일에서 받는다 (2026-11-11 사용자 결정)

Mac 시스템 승인창(`request_access`, `request_full_control`)은 모바일에 안 뜨고 `AskUserQuestion` 은 뜬다.

- 앱 제어·전체 화면 제어가 필요하면 먼저 질문 창으로 무엇을 왜 하는지(앱·할 일·보낼 내용) 묻고, 승인 뒤 Mac 승인창을 띄우며 Mac 에서 눌러야 한다고 알린다.
- 메시지·파일 전송처럼 되돌리기 어려운 동작도 같은 방식으로 확인한다(사용자가 내용까지 지시한 경우는 제외).
- 전체 화면 제어는 백그라운드(`app_*`)로 안 될 때만 쓴다(다른 Space·전체 화면 앱이 앞에 있으면 막힌다).
