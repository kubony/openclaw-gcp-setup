# 4. OpenClaw 설치 및 설정 가이드

이 문서는 Ubuntu VM에서 OpenClaw를 설치하고 텔레그램 봇까지 연동하는 전체 과정을 안내합니다.

---

## 용어 설명

| 용어 | 설명 |
|------|------|
| **OpenClaw** | Claude AI를 메신저(텔레그램 등)에서 사용할 수 있게 해주는 봇 |
| **Node.js** | JavaScript를 실행하는 환경. OpenClaw가 이걸로 만들어짐 |
| **pnpm** | 패키지 관리자. npm보다 빠르고 효율적 |
| **게이트웨이** | OpenClaw의 백그라운드 서비스. 메시지를 받고 처리함 |
| **Systemd** | Linux의 서비스 관리자. 프로그램을 백그라운드에서 실행 |

---

## 목차

1. [사전 준비](#1-사전-준비)
2. [의존성 설치](#2-의존성-설치) - Node.js, pnpm
3. [OpenClaw 설치](#3-openclaw-설치) - 클론, 빌드
4. [초기 설정](#4-초기-설정-onboarding) - Anthropic 토큰 연결
5. [백그라운드 서비스](#5-백그라운드-서비스-설정) - Systemd
6. [텔레그램 봇 연동](#6-텔레그램-봇-연동) - BotFather, Pairing
7. [실행 확인](#7-실행-확인)

예상 소요 시간: 20-30분

---

## 1. 사전 준비

### 필요한 것

- [ ] Ubuntu VM (22.04 이상) - [이전 문서](03-vm-create.md)에서 생성
- [ ] SSH로 VM 접속된 상태
- [ ] Claude 계정 (Anthropic API 또는 Claude Pro)

### VM 접속 확인

```bash
# 로컬 터미널에서 VM 접속
gcloud compute ssh openclaw-vm --zone=asia-northeast3-a
```

접속되면 프롬프트가 바뀝니다:
```
username@openclaw-vm:~$
```

---

## 2. 의존성 설치

### 2.1 Node.js 22.x 설치

```bash
# NodeSource 저장소 추가 및 Node.js 설치
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
```

> **curl | bash가 걱정되시나요?**
>
> 이 명령어는 NodeSource(Node.js 공식 배포처)의 스크립트를 실행합니다.
> - NodeSource는 Node.js 재단과 파트너십을 맺은 공식 배포자
> - 수백만 서버에서 사용되는 표준 설치 방법
> - 걱정된다면 [스크립트 내용](https://deb.nodesource.com/setup_22.x)을 먼저 확인해보세요

**설치 확인**:
```bash
node --version
```

**정상 출력**: `v22.x.x` (숫자는 다를 수 있음)

### 2.2 pnpm 설치

```bash
# pnpm 설치
sudo npm install -g pnpm

# pnpm 경로 설정
pnpm setup
```

> **⚠️ 중요: 새 터미널이 필요합니다!**
>
> `pnpm setup` 실행 후 PATH가 업데이트됩니다.
> 다음 중 하나를 실행하세요:
> ```bash
> # 방법 1: 현재 터미널에서 설정 다시 로드
> source ~/.bashrc
>
> # 방법 2: 또는 SSH 재접속
> exit
> gcloud compute ssh openclaw-vm --zone=asia-northeast3-a
> ```

**설치 확인**:
```bash
pnpm --version
```

**정상 출력**: `10.x.x` 이상

---

## 3. OpenClaw 설치

### 3.1 소스 코드 다운로드

```bash
cd ~
git clone https://github.com/anthropics/openclaw
cd openclaw
```

### 3.2 빌드

```bash
# 의존성 설치 (시간이 좀 걸림)
pnpm install

# 빌드
pnpm build
```

**정상 완료 시**: 에러 없이 프롬프트로 돌아옴

### 3.3 글로벌 설치

어디서든 `openclaw` 명령어를 사용하려면:

```bash
sudo npm install -g .
```

**확인**:
```bash
which openclaw
openclaw --version
```

**정상 출력**:
```
/usr/bin/openclaw
2026.x.x
```

---

## 4. 초기 설정 (Onboarding)

### 4.1 Onboarding 실행

```bash
cd ~/openclaw
pnpm openclaw onboard
```

### 4.2 설정 마법사 따라하기

대화형 설정 마법사가 실행됩니다:

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  Step 1: Security warning                       │
│  ─────────────────────                          │
│  OpenClaw는 시스템에 접근합니다. 계속할까요?   │
│                                                 │
│  > Yes                                          │
│    No                                           │
│                                                 │
└─────────────────────────────────────────────────┘
```
→ **Yes** 선택

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  Step 2: Onboarding mode                        │
│  ─────────────────────                          │
│  > QuickStart (권장)                            │
│    Advanced                                     │
│                                                 │
└─────────────────────────────────────────────────┘
```
→ **QuickStart** 선택

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  Step 3: Model/Auth provider                    │
│  ─────────────────────                          │
│  > Anthropic                                    │
│    OpenAI                                       │
│    Azure                                        │
│                                                 │
└─────────────────────────────────────────────────┘
```
→ **Anthropic** 선택

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  Step 4: Anthropic auth method                  │
│  ─────────────────────                          │
│  > Anthropic token (paste setup-token)          │
│    API key                                      │
│                                                 │
└─────────────────────────────────────────────────┘
```
→ **Anthropic token** 선택

### 4.3 Setup Token 얻기

**새 터미널을 열어서** (로컬 컴퓨터에서):

```bash
# Claude Code가 설치된 로컬에서 실행
claude setup-token
```

출력된 토큰을 복사해서 VM 터미널에 붙여넣기

### 4.4 설정 확인

설정이 `~/.openclaw/openclaw.json`에 저장됩니다:

```bash
cat ~/.openclaw/openclaw.json | head -20
```

---

## 5. 백그라운드 서비스 설정

OpenClaw 게이트웨이를 24시간 실행하려면 Systemd 서비스를 설정합니다.

### 5.1 자동 설정 스크립트 (권장)

아래 명령어 한 줄로 서비스 파일을 자동 생성합니다:

```bash
# 사용자명과 토큰 자동 감지
USERNAME=$(whoami)
TOKEN=$(cat ~/.openclaw/openclaw.json | grep -o '"token": "[^"]*"' | head -1 | cut -d'"' -f4)

# 서비스 디렉토리 생성
mkdir -p ~/.config/systemd/user/

# 서비스 파일 생성
cat > ~/.config/systemd/user/openclaw-gateway.service << EOF
[Unit]
Description=OpenClaw Gateway
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/bin/node /home/${USERNAME}/openclaw/dist/index.js gateway --port 18789
Restart=always
RestartSec=5
KillMode=process
Environment=HOME=/home/${USERNAME}
Environment=PATH=/home/${USERNAME}/.local/bin:/home/${USERNAME}/.local/share/pnpm:/usr/local/bin:/usr/bin:/bin
Environment=OPENCLAW_GATEWAY_PORT=18789
Environment=OPENCLAW_GATEWAY_TOKEN=${TOKEN}
Environment=OPENCLAW_SYSTEMD_UNIT=openclaw-gateway.service
Environment=OPENCLAW_SERVICE_MARKER=openclaw
Environment=OPENCLAW_SERVICE_KIND=gateway

[Install]
WantedBy=default.target
EOF

echo "서비스 파일 생성 완료!"
```

### 5.2 서비스 시작

```bash
# systemd 설정 다시 로드
systemctl --user daemon-reload

# 서비스 활성화 (부팅 시 자동 시작)
systemctl --user enable openclaw-gateway

# 서비스 시작
systemctl --user start openclaw-gateway

# 상태 확인
systemctl --user status openclaw-gateway
```

**정상 출력**:
```
● openclaw-gateway.service - OpenClaw Gateway
     Loaded: loaded
     Active: active (running) since ...
```

> **GATEWAY_TOKEN을 수동으로 찾으려면?**
>
> ```bash
> cat ~/.openclaw/openclaw.json | grep -A2 '"gateway"' | grep token
> ```
> 또는 JSON 파일을 직접 열어서 `gateway.auth.token` 값 확인

### 5.3 서비스 관리 명령어

```bash
# 재시작
systemctl --user restart openclaw-gateway

# 중지
systemctl --user stop openclaw-gateway

# 로그 보기
journalctl --user -u openclaw-gateway -f
```

---

## 6. 텔레그램 봇 연동

### 6.1 BotFather에서 봇 생성

1. **텔레그램 앱**을 엽니다 (폰 또는 PC)

2. 검색창에 **@BotFather** 입력하고 선택
   ```
   ┌─────────────────────────────────┐
   │ 🔍 @BotFather                   │
   │                                 │
   │ BotFather                       │
   │ @BotFather ✓ (파란 체크 확인)  │
   │ The Official Bot Father         │
   └─────────────────────────────────┘
   ```

3. **대화 시작** 버튼 클릭

4. `/newbot` 입력하고 전송

5. **봇 이름** 입력 (표시되는 이름)
   ```
   예: My OpenClaw Bot
   ```

6. **봇 username** 입력 (고유 ID)
   ```
   예: my_openclaw_bot

   ⚠️ 반드시 _bot으로 끝나야 함!
   ⚠️ 이미 사용 중이면 다른 이름 시도
   ```

7. **Bot Token** 복사
   ```
   BotFather가 보내는 메시지에서:

   Done! Congratulations on your new bot...
   Use this token to access the HTTP API:
   1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ  ← 이거 복사!
   ```

### 6.2 OpenClaw에 봇 연결

**방법 A**: Onboarding에서 설정 (아직 안 했다면)

```bash
pnpm openclaw onboard
```
→ 채널 선택에서 Telegram 선택 → Bot Token 붙여넣기

**방법 B**: 설정 파일 직접 수정

```bash
# 설정 파일 열기
nano ~/.openclaw/openclaw.json
```

다음 내용 추가/수정:
```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "botToken": "여기에_복사한_토큰_붙여넣기"
    }
  }
}
```

저장: `Ctrl+O` → `Enter` → `Ctrl+X`

### 6.3 게이트웨이 재시작

```bash
systemctl --user restart openclaw-gateway
```

### 6.4 Pairing (본인 인증)

1. **텔레그램에서 내 봇 찾기**
   - @my_openclaw_bot (본인이 만든 username) 검색

2. **아무 메시지나 보내기**
   ```
   안녕
   ```

3. **봇이 pairing 코드 응답**:
   ```
   OpenClaw: access not configured.
   Your Telegram user id: 12345678
   Pairing code: ABC12345
   ```

4. **VM에서 승인 명령어 실행**:
   ```bash
   openclaw pairing approve telegram ABC12345
   ```

5. **승인 완료!** 이제 봇에게 다시 메시지 보내보세요:
   ```
   안녕하세요!
   ```
   → Claude가 응답합니다

---

## 7. 실행 확인

### 7.1 게이트웨이 상태

```bash
systemctl --user status openclaw-gateway
```

정상이면 `active (running)` 표시

### 7.2 채널 상태

```bash
openclaw channels status
```

정상이면 telegram이 `connected` 표시

### 7.3 TUI (선택)

터미널 UI로 상태 확인:

```bash
openclaw tui
```

종료: `Ctrl+C`

### 7.4 로그 확인

```bash
# 실시간 로그
journalctl --user -u openclaw-gateway -f

# 최근 50줄
journalctl --user -u openclaw-gateway -n 50
```

---

## 문제 해결

### "pnpm: command not found"

```bash
source ~/.bashrc
# 또는 SSH 재접속
```

### "openclaw: command not found"

```bash
cd ~/openclaw
sudo npm install -g .
```

### 게이트웨이가 시작되지 않음

```bash
# 로그 확인
journalctl --user -u openclaw-gateway -n 50

# 포트 사용 중인지 확인
ss -tlnp | grep 18789

# 수동 실행 테스트
cd ~/openclaw
node dist/index.js gateway --port 18789
```

### 텔레그램 봇이 응답 안 함

1. Bot Token이 올바른지 확인
2. 게이트웨이가 실행 중인지 확인
3. Pairing 승인했는지 확인

```bash
# pairing 상태 확인
openclaw pairing list
```

### "Permission denied" 에러

```bash
sudo npm install -g .
```

---

## 체크리스트

모든 설정이 완료되었는지 확인:

- [ ] `node --version` → v22.x.x
- [ ] `pnpm --version` → 10.x.x
- [ ] `openclaw --version` → 버전 출력됨
- [ ] `systemctl --user status openclaw-gateway` → active (running)
- [ ] 텔레그램 봇에게 메시지 → Claude 응답

---

## 설정 파일 위치

| 파일 | 경로 | 용도 |
|------|------|------|
| OpenClaw 설정 | `~/.openclaw/openclaw.json` | 모든 설정 |
| 워크스페이스 | `~/.openclaw/workspace/` | 작업 파일 |
| Systemd 서비스 | `~/.config/systemd/user/openclaw-gateway.service` | 백그라운드 서비스 |
| 소스 코드 | `~/openclaw/` | OpenClaw 코드 |

---

## 버전 정보

| 구성요소 | 최소 버전 | 확인 명령어 |
|----------|-----------|-------------|
| Node.js | v22.x 이상 | `node --version` |
| pnpm | 10.x 이상 | `pnpm --version` |
| OpenClaw | 최신 권장 | `openclaw --version` |

> **업데이트 방법**:
> ```bash
> cd ~/openclaw
> git pull
> pnpm install
> pnpm build
> sudo npm install -g .
> systemctl --user restart openclaw-gateway
> ```

---

*최종 업데이트: 2026-02-01*
