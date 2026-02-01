# 2. Google Cloud SDK 설치 및 인증

이 문서는 로컬 컴퓨터에서 GCP를 제어하기 위한 Google Cloud SDK(gcloud CLI) 설치 방법을 안내합니다.

---

## 용어 설명

| 용어 | 설명 |
|------|------|
| **gcloud** | GCP를 명령어로 제어하는 도구. "지클라우드"라고 읽음 |
| **CLI** | Command Line Interface. 터미널에서 명령어로 조작하는 방식 |
| **SDK** | Software Development Kit. 개발에 필요한 도구 모음 |
| **터미널** | 명령어를 입력하는 프로그램 (macOS: Terminal, Windows: PowerShell) |

---

## 설치 방법 선택

| OS | 권장 방법 | 난이도 |
|----|----------|--------|
| macOS | Homebrew | ★☆☆ 쉬움 |
| Ubuntu/Debian | apt 패키지 | ★★☆ 보통 |
| Windows | 설치 프로그램 | ★☆☆ 쉬움 |

---

## macOS 설치

### 사전 준비: Homebrew 설치

Homebrew가 없다면 먼저 설치하세요:

```bash
# Homebrew 설치 (한 줄 복사해서 터미널에 붙여넣기)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 설치 확인
brew --version
```

> **Homebrew가 뭔가요?**
> macOS용 패키지 관리자입니다. 앱스토어처럼 명령어로 프로그램을 설치/삭제할 수 있어요.

### gcloud 설치

```bash
# Homebrew로 gcloud 설치
brew install --cask google-cloud-sdk

# 설치 확인
gcloud --version
```

**정상 출력 예시:**
```
Google Cloud SDK 460.0.0
bq 2.0.101
core 2024.01.26
gcloud-crc32c 1.0.0
gsutil 5.27
```

버전 숫자가 다르더라도 출력이 나오면 성공입니다!

---

## Ubuntu/Debian 설치

### 방법 1: 한 줄 설치 스크립트 (권장)

아래 명령어를 **한 줄씩** 복사해서 실행하세요:

```bash
# 1. 필요한 패키지 설치
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates gnupg curl

# 2. Google Cloud 공개 키 추가
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg

# 3. 저장소 추가
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

# 4. gcloud 설치
sudo apt-get update
sudo apt-get install -y google-cloud-cli

# 5. 설치 확인
gcloud --version
```

> **명령어가 복잡해 보이나요?**
>
> 걱정 마세요, 한 번만 실행하면 됩니다:
> - 1-2번: 설치 준비
> - 3번: Google 저장소 등록
> - 4번: 실제 설치
> - 5번: 확인

### 방법 2: 공식 스크립트

```bash
# 설치 스크립트 다운로드 및 실행
curl https://sdk.cloud.google.com | bash

# 셸 재시작 (중요!)
exec -l $SHELL

# 설치 확인
gcloud --version
```

---

## Windows 설치

1. [Google Cloud SDK 설치 프로그램](https://cloud.google.com/sdk/docs/install) 다운로드

2. 다운로드된 `GoogleCloudSDKInstaller.exe` 실행

3. 설치 마법사 따라 진행 (기본 옵션 유지)

4. 설치 완료 후 **"Google Cloud SDK Shell"** 실행
   - 시작 메뉴에서 "Google Cloud SDK Shell" 검색

5. 설치 확인:
   ```
   gcloud --version
   ```

---

## 인증 설정

### Step 1: Google 계정 로그인

```bash
gcloud auth login
```

**무슨 일이 일어나나요?**

1. 브라우저가 자동으로 열림
2. Google 계정 선택 화면 표시
3. GCP 접근 권한 허용 클릭
4. "인증이 완료되었습니다" 메시지 확인
5. 터미널로 돌아오면 완료!

> **브라우저가 안 열리나요? (서버/VM 환경)**
>
> 화면이 없는 환경에서는 다음 명령어 사용:
> ```bash
> gcloud auth login --no-launch-browser
> ```
> 터미널에 표시되는 URL을 복사해서 다른 컴퓨터 브라우저에 붙여넣기

### Step 2: 기본 프로젝트 설정

```bash
# 내 프로젝트 목록 확인
gcloud projects list
```

**출력 예시:**
```
PROJECT_ID                  NAME                PROJECT_NUMBER
my-openclaw-project-12345   My OpenClaw         123456789012
```

`PROJECT_ID` 열의 값을 복사해서:

```bash
# 기본 프로젝트 설정 (자신의 프로젝트 ID로 변경!)
gcloud config set project my-openclaw-project-12345
```

> **프로젝트 ID를 모르겠다면?**
>
> [이전 문서](01-gcp-account.md)의 "프로젝트 ID 확인하는 법" 섹션 참고

### Step 3: 기본 리전/존 설정

```bash
# 서울 리전으로 설정 (한국에서 가장 빠름)
gcloud config set compute/region asia-northeast3
gcloud config set compute/zone asia-northeast3-a
```

### Step 4: 설정 확인

```bash
gcloud config list
```

**정상 출력 예시:**
```
[compute]
region = asia-northeast3
zone = asia-northeast3-a
[core]
account = your-email@gmail.com
project = my-openclaw-project-12345
```

---

## 추가 인증 (Compute Engine API용)

VM 생성 등을 하려면 추가 인증이 필요합니다:

```bash
gcloud auth application-default login
```

브라우저가 열리면 다시 한번 권한 허용하면 됩니다.

---

## 자주 사용하는 명령어

| 명령어 | 설명 | 언제 사용? |
|--------|------|-----------|
| `gcloud auth login` | 계정 로그인 | 처음 설정 시 |
| `gcloud auth list` | 로그인된 계정 목록 | 계정 확인 시 |
| `gcloud config list` | 현재 설정 확인 | 설정 확인 시 |
| `gcloud config set project ID` | 프로젝트 변경 | 다른 프로젝트 사용 시 |
| `gcloud projects list` | 프로젝트 목록 | 프로젝트 ID 확인 시 |
| `gcloud compute instances list` | VM 목록 | VM 확인 시 |
| `gcloud components update` | SDK 업데이트 | 가끔 |

---

## 문제 해결

### "gcloud: command not found"

**원인**: gcloud가 PATH에 없음

**해결 (macOS/Linux)**:
```bash
# 셸 설정 파일에 추가
echo 'source ~/google-cloud-sdk/path.bash.inc' >> ~/.bashrc
source ~/.bashrc

# 또는 새 터미널 열기
```

**해결 (macOS with Homebrew)**:
```bash
# Homebrew로 설치했다면 자동 설정됨
# 새 터미널을 열어보세요
```

### "You do not have permission"

**원인**: 권한 없음 또는 잘못된 계정

**해결**:
```bash
# 현재 어떤 계정인지 확인
gcloud auth list

# 활성 계정에 * 표시됨
# 다른 계정으로 전환하려면:
gcloud config set account other-email@gmail.com

# 또는 다시 로그인:
gcloud auth login
```

### "API has not been enabled"

**원인**: 필요한 API가 비활성화됨

**해결**:
```bash
# Compute Engine API 활성화
gcloud services enable compute.googleapis.com

# 활성화된 API 목록 확인
gcloud services list --enabled
```

### "Could not open browser"

**원인**: 화면이 없는 환경 (서버, VM 등)

**해결**:
```bash
# 브라우저 없이 인증
gcloud auth login --no-launch-browser

# 표시되는 URL을 다른 컴퓨터에서 열고
# 인증 코드를 복사해서 터미널에 붙여넣기
```

---

## 체크리스트

다음 단계로 넘어가기 전에 확인하세요:

- [ ] `gcloud --version` 실행 시 버전 정보 출력됨
- [ ] `gcloud auth list`에서 내 계정이 보임
- [ ] `gcloud config list`에서 project가 설정됨
- [ ] `gcloud config list`에서 region/zone이 asia-northeast3로 설정됨

---

## 다음 단계

gcloud CLI 설치와 인증이 완료되었으면 다음 문서로 진행하세요:

→ [03. GCP VM 생성](03-vm-create.md)

---

*최종 업데이트: 2026-02-01*
