# 2. Google Cloud SDK 설치 및 인증

이 문서는 로컬 컴퓨터에서 GCP를 제어하기 위한 Google Cloud SDK(gcloud CLI) 설치 방법을 안내합니다.

---

## 설치 방법 선택

| OS | 권장 방법 |
|----|----------|
| macOS | Homebrew 또는 공식 설치 스크립트 |
| Ubuntu/Debian | apt 패키지 또는 공식 설치 스크립트 |
| Windows | 공식 설치 프로그램 |

---

## macOS 설치

### 방법 1: Homebrew (권장)

```bash
# Homebrew로 설치
brew install --cask google-cloud-sdk

# 설치 확인
gcloud --version
```

### 방법 2: 공식 스크립트

```bash
# 설치 스크립트 다운로드 및 실행
curl https://sdk.cloud.google.com | bash

# 새 터미널 열거나 셸 재시작
exec -l $SHELL

# 설치 확인
gcloud --version
```

---

## Ubuntu/Debian 설치

### 방법 1: apt 패키지 (권장)

```bash
# Google Cloud 공개 키 추가
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg

# 저장소 추가
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

# 패키지 업데이트 및 설치
sudo apt-get update && sudo apt-get install google-cloud-cli

# 설치 확인
gcloud --version
```

### 방법 2: 공식 스크립트

```bash
# 설치 스크립트 다운로드 및 실행
curl https://sdk.cloud.google.com | bash

# 셸 재시작
exec -l $SHELL

# 설치 확인
gcloud --version
```

---

## Windows 설치

1. [Google Cloud SDK 설치 프로그램](https://cloud.google.com/sdk/docs/install) 다운로드
2. 설치 프로그램 실행
3. 설치 완료 후 "Google Cloud SDK Shell" 실행
4. `gcloud --version`으로 확인

---

## 인증 설정

### Step 1: Google 계정 로그인

```bash
gcloud auth login
```

1. 브라우저가 열리면 Google 계정 선택
2. GCP 접근 권한 허용
3. "인증이 완료되었습니다" 메시지 확인

### Step 2: 기본 프로젝트 설정

```bash
# 프로젝트 목록 확인
gcloud projects list

# 기본 프로젝트 설정
gcloud config set project <YOUR_PROJECT_ID>
```

### Step 3: 기본 리전/존 설정 (선택)

```bash
# 서울 리전으로 설정
gcloud config set compute/region asia-northeast3
gcloud config set compute/zone asia-northeast3-a
```

### Step 4: 설정 확인

```bash
gcloud config list
```

출력 예시:
```
[compute]
region = asia-northeast3
zone = asia-northeast3-a
[core]
account = your-email@gmail.com
project = your-project-id
```

---

## 추가 인증 (서비스 계정용)

VM 생성 등 Compute Engine API를 사용하려면 Application Default Credentials가 필요합니다:

```bash
gcloud auth application-default login
```

---

## 자주 사용하는 명령어

| 명령어 | 설명 |
|--------|------|
| `gcloud auth login` | Google 계정 로그인 |
| `gcloud auth list` | 로그인된 계정 목록 |
| `gcloud config list` | 현재 설정 확인 |
| `gcloud projects list` | 프로젝트 목록 |
| `gcloud compute instances list` | VM 목록 |
| `gcloud components update` | SDK 업데이트 |

---

## 문제 해결

### "gcloud: command not found"

셸 PATH에 gcloud가 없습니다:

```bash
# bashrc에 PATH 추가 (설치 위치에 따라 다름)
echo 'source ~/google-cloud-sdk/path.bash.inc' >> ~/.bashrc
source ~/.bashrc
```

### "You do not have permission"

프로젝트 권한이 없거나 잘못된 계정:

```bash
# 현재 계정 확인
gcloud auth list

# 다른 계정으로 전환
gcloud config set account <OTHER_ACCOUNT>
```

### API가 활성화되지 않음

```bash
# Compute Engine API 활성화
gcloud services enable compute.googleapis.com
```

---

## 다음 단계

gcloud CLI 설치와 인증이 완료되었으면 다음 문서로 진행하세요:

→ [03. GCP VM 생성](03-vm-create.md)

---

*최종 업데이트: 2026-02-01*
