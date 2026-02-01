# OpenClaw 설치 및 설정 가이드

이 문서는 Ubuntu VM에서 OpenClaw를 설치하고 설정하는 전체 과정을 기록합니다.

## 목차
1. [사전 준비](#1-사전-준비)
2. [OpenClaw 소스 클론](#2-openclaw-소스-클론)
3. [의존성 설치](#3-의존성-설치)
4. [OpenClaw 빌드](#4-openclaw-빌드)
5. [Onboarding (초기 설정)](#5-onboarding-초기-설정)
6. [글로벌 설치](#6-글로벌-설치)
7. [Systemd 서비스 설정](#7-systemd-서비스-설정)
8. [Telegram 채널 설정](#8-telegram-채널-설정)
9. [실행 및 확인](#9-실행-및-확인)

---

## 1. 사전 준비

### 시스템 요구사항
- Ubuntu Linux (22.04 LTS 이상)
- 최소 2GB RAM
- 인터넷 연결

---

## 2. OpenClaw 소스 클론

```bash
cd ~
git clone https://github.com/openclaw/openclaw
cd openclaw
```

---

## 3. 의존성 설치

### 3.1 Node.js 22.x 설치

```bash
# NodeSource에서 Node.js 22.x 설치
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# 설치 확인
node --version  # v22.22.0 이상
```

### 3.2 pnpm 설치

```bash
# pnpm 글로벌 설치
sudo npm install -g pnpm@10.23.0

# pnpm setup (PATH 설정)
pnpm setup

# bashrc 재로드 또는 새 터미널 열기
source ~/.bashrc

# 설치 확인
pnpm --version  # 10.23.0
```

> **참고**: `pnpm setup` 실행 후 `~/.bashrc`에 다음이 추가됩니다:
> ```bash
> # pnpm
> export PNPM_HOME="/home/<YOUR_USERNAME>/.local/share/pnpm"
> case ":$PATH:" in
>   *":$PNPM_HOME:"*) ;;
>   *) export PATH="$PNPM_HOME:$PATH" ;;
> esac
> # pnpm end
> ```

---

## 4. OpenClaw 빌드

```bash
cd ~/openclaw

# 의존성 설치
pnpm install

# 빌드
pnpm build
```

---

## 5. Onboarding (초기 설정)

### 5.1 Onboarding 실행

```bash
cd ~/openclaw
pnpm openclaw onboard
```

### 5.2 설정 선택

대화형 설정 마법사가 실행됩니다. 다음과 같이 선택하세요:

1. **Security warning 확인**: `Yes` 선택
2. **Onboarding mode**: `QuickStart` 선택
3. **Config handling**: `Use existing values` (기존 설정이 있으면) 또는 새로 설정
4. **Model/auth provider**: `Anthropic` 선택
5. **Anthropic auth method**: `Anthropic token (paste setup-token)` 선택
6. **Anthropic setup-token**:
   - 별도 터미널에서 `claude setup-token` 실행
   - 생성된 토큰 붙여넣기

### 5.3 기본 설정값 (참고)

설정 완료 후 `~/.openclaw/openclaw.json`에 저장됩니다:

| 항목 | 값 |
|------|-----|
| workspace | `~/.openclaw/workspace` |
| gateway.mode | `local` |
| gateway.port | `18789` |
| gateway.bind | `loopback` (127.0.0.1) |
| skills.nodeManager | `pnpm` |

---

## 6. 글로벌 설치

`openclaw` 명령어를 어디서든 사용하려면 글로벌 설치가 필요합니다:

```bash
cd ~/openclaw
sudo npm install -g .

# 설치 확인
which openclaw      # /usr/bin/openclaw
openclaw --version  # 설치된 버전 출력
```

---

## 7. Systemd 서비스 설정

게이트웨이를 백그라운드 서비스로 실행하려면:

### 7.1 서비스 파일 생성

```bash
mkdir -p ~/.config/systemd/user/
```

`~/.config/systemd/user/openclaw-gateway.service` 파일 생성:

```ini
[Unit]
Description=OpenClaw Gateway
After=network-online.target
Wants=network-online.target

[Service]
ExecStart="/usr/bin/node" "/home/<YOUR_USERNAME>/openclaw/dist/index.js" gateway --port 18789
Restart=always
RestartSec=5
KillMode=process
Environment=HOME=/home/<YOUR_USERNAME>
Environment="PATH=/home/<YOUR_USERNAME>/.local/bin:/home/<YOUR_USERNAME>/.local/share/pnpm:/usr/local/bin:/usr/bin:/bin"
Environment=OPENCLAW_GATEWAY_PORT=18789
Environment=OPENCLAW_GATEWAY_TOKEN=<YOUR_GATEWAY_TOKEN>
Environment="OPENCLAW_SYSTEMD_UNIT=openclaw-gateway.service"
Environment=OPENCLAW_SERVICE_MARKER=openclaw
Environment=OPENCLAW_SERVICE_KIND=gateway

[Install]
WantedBy=default.target
```

> **주의**: 다음 값들을 실제 환경에 맞게 변경하세요:
> - `<YOUR_USERNAME>`: 실제 사용자명 (예: `ubuntu`, `user1`)
> - `<YOUR_GATEWAY_TOKEN>`: `~/.openclaw/openclaw.json`의 `gateway.auth.token` 값

### 7.2 서비스 활성화 및 시작

```bash
# systemd 데몬 리로드
systemctl --user daemon-reload

# 서비스 활성화 (부팅 시 자동 시작)
systemctl --user enable openclaw-gateway

# 서비스 시작
systemctl --user start openclaw-gateway

# 상태 확인
systemctl --user status openclaw-gateway
```

### 7.3 서비스 재시작

```bash
systemctl --user restart openclaw-gateway
```

---

## 8. Telegram 채널 설정

### 8.1 Telegram Bot 생성

먼저 Telegram에서 봇을 생성합니다:

1. Telegram 앱에서 [@BotFather](https://t.me/BotFather) 검색하여 대화 시작
2. `/newbot` 명령어 입력
3. 봇 이름 입력 (예: `My OpenClaw Bot`)
4. 봇 username 입력 (예: `my_openclaw_bot` - 반드시 `_bot`으로 끝나야 함)
5. **Bot Token** 복사하여 안전한 곳에 저장 (예: `1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ`)

### 8.2 Telegram 채널 설정

**방법 A**: Onboarding 중 설정 (권장)

`pnpm openclaw onboard` 실행 시 채널 선택에서 Telegram을 선택하면 Bot Token 입력 프롬프트가 나타납니다.

**방법 B**: 수동 설정

이미 Onboarding을 완료한 경우, `~/.openclaw/openclaw.json`에 직접 추가:

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "botToken": "<YOUR_BOT_TOKEN>",
      "groupPolicy": "allowlist",
      "streamMode": "partial",
      "mediaMaxMb": 20
    }
  },
  "plugins": {
    "entries": {
      "telegram": {
        "enabled": true
      }
    }
  }
}
```

### 8.3 Pairing 승인

봇에게 첫 메시지를 보내면 pairing 코드를 받습니다:

```
OpenClaw: access not configured.
Your Telegram user id: 40277226
Pairing code: Q5EWM7R7
```

승인 명령어 실행:

```bash
pnpm openclaw pairing approve telegram Q5EWM7R7
# 또는 글로벌 설치 후:
openclaw pairing approve telegram Q5EWM7R7
```

---

## 9. 실행 및 확인

### 9.1 TUI (Terminal UI) 실행

```bash
openclaw tui
```

TUI에서 확인할 수 있는 정보:
- 연결 상태: `ws://127.0.0.1:18789`
- 에이전트: `agent main`
- 세션: `session main`
- 모델: `anthropic/claude-opus-4-5`
- 상태: `connected | idle`

### 9.2 채널 상태 확인

```bash
openclaw channels status
```

### 9.3 게이트웨이 로그 확인

```bash
# Systemd 로그
journalctl --user -u openclaw-gateway -f

# 또는 파일 로그
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

---

## 추가 스킬 설정 (선택)

`~/.openclaw/openclaw.json`의 `skills.entries`에 API 키 추가:

```json
{
  "skills": {
    "entries": {
      "nano-banana-pro": {
        "apiKey": "<GOOGLE_AI_API_KEY>"
      },
      "openai-image-gen": {
        "apiKey": "<OPENAI_API_KEY>"
      },
      "openai-whisper-api": {
        "apiKey": "<OPENAI_API_KEY>"
      }
    }
  }
}
```

---

## 문제 해결

### `openclaw: command not found`

글로벌 설치 필요:
```bash
cd ~/openclaw
sudo npm install -g .
```

### 권한 오류 (EACCES)

sudo 사용:
```bash
sudo npm install -g .
```

### 게이트웨이 연결 실패

1. 서비스 상태 확인: `systemctl --user status openclaw-gateway`
2. 포트 확인: `ss -tlnp | grep 18789`
3. 로그 확인: `journalctl --user -u openclaw-gateway -n 50`

---

## 버전 정보

| 구성요소 | 최소 버전 | 확인 명령어 |
|----------|-----------|-------------|
| Node.js | v22.x 이상 | `node --version` |
| pnpm | 10.x 이상 | `pnpm --version` |
| OpenClaw | 최신 권장 | `openclaw --version` |

> **팁**: OpenClaw 최신 버전은 [GitHub Releases](https://github.com/anthropics/claude-code/releases)에서 확인하세요.

---

## 설정 파일 위치

| 파일 | 경로 |
|------|------|
| OpenClaw 설정 | `~/.openclaw/openclaw.json` |
| 워크스페이스 | `~/.openclaw/workspace/` |
| Systemd 서비스 | `~/.config/systemd/user/openclaw-gateway.service` |
| 소스 코드 | `~/openclaw/` |

---

*최종 업데이트: 2026-02-01*
