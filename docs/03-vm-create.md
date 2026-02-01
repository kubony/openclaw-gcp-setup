# 3. GCP VM 생성

이 문서는 OpenClaw를 실행할 GCP VM(가상 머신)을 생성하는 방법을 안내합니다.

---

## 권장 사양

OpenClaw 실행을 위한 권장 VM 사양:

| 항목 | 최소 | 권장 | 비고 |
|------|------|------|------|
| vCPU | 2 | 4 | 동시 작업 시 4 이상 권장 |
| RAM | 4GB | 8GB | 여러 에이전트 실행 시 |
| 디스크 | 20GB | 50GB | SSD 권장 |
| OS | Ubuntu 22.04 | Ubuntu 24.04 | LTS 버전 권장 |

---

## 방법 1: gcloud CLI로 생성 (권장)

### Step 1: API 활성화

```bash
gcloud services enable compute.googleapis.com
```

### Step 2: VM 생성

```bash
gcloud compute instances create openclaw-vm \
  --project=<YOUR_PROJECT_ID> \
  --zone=asia-northeast3-a \
  --machine-type=e2-standard-4 \
  --image-family=ubuntu-2404-lts-amd64 \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-ssd \
  --tags=http-server,https-server
```

> **변경 가능한 값**:
> - `openclaw-vm`: VM 이름
> - `<YOUR_PROJECT_ID>`: 프로젝트 ID
> - `e2-standard-4`: 머신 타입 (아래 표 참고)

### 머신 타입 선택 가이드

| 머신 타입 | vCPU | RAM | 월 비용 | 용도 |
|-----------|------|-----|---------|------|
| e2-small | 2 | 2GB | ~$15 | 테스트용 |
| e2-medium | 2 | 4GB | ~$25 | 최소 사양 |
| e2-standard-4 | 4 | 16GB | ~$100 | **권장** |
| e2-standard-8 | 8 | 32GB | ~$200 | 고성능 |

### Step 3: 방화벽 규칙 확인

기본적으로 SSH(22번 포트)는 열려 있습니다. 추가 포트가 필요하면:

```bash
# 예: 8080 포트 열기
gcloud compute firewall-rules create allow-8080 \
  --allow=tcp:8080 \
  --target-tags=http-server
```

---

## 방법 2: GCP Console에서 생성

### Step 1: Console 접속

1. [GCP Console](https://console.cloud.google.com) 접속
2. 좌측 메뉴 → **Compute Engine** → **VM 인스턴스**

### Step 2: 인스턴스 만들기

1. **"인스턴스 만들기"** 클릭
2. 다음 설정 입력:

| 항목 | 값 |
|------|-----|
| 이름 | `openclaw-vm` |
| 리전 | `asia-northeast3 (서울)` |
| 영역 | `asia-northeast3-a` |
| 머신 유형 | `e2-standard-4` |

3. **부팅 디스크** → **변경** 클릭:
   - 운영체제: Ubuntu
   - 버전: Ubuntu 24.04 LTS
   - 디스크 유형: SSD 영구 디스크
   - 크기: 50GB

4. **만들기** 클릭

---

## SSH 접속

### 방법 1: gcloud 명령어

```bash
gcloud compute ssh openclaw-vm --zone=asia-northeast3-a
```

처음 접속 시 SSH 키가 자동 생성됩니다.

### 방법 2: 일반 SSH

VM의 외부 IP 확인:

```bash
gcloud compute instances describe openclaw-vm \
  --zone=asia-northeast3-a \
  --format='get(networkInterfaces[0].accessConfigs[0].natIP)'
```

접속:

```bash
ssh -i ~/.ssh/google_compute_engine <YOUR_USERNAME>@<EXTERNAL_IP>
```

### 방법 3: ~/.ssh/config 설정 (권장)

반복 접속을 편하게 하려면 `~/.ssh/config`에 추가:

```
Host openclaw-vm
  HostName <EXTERNAL_IP>
  User <YOUR_USERNAME>
  IdentityFile ~/.ssh/google_compute_engine
```

이후 간단히:

```bash
ssh openclaw-vm
```

---

## VM 초기 설정

SSH 접속 후 기본 패키지 업데이트:

```bash
# 패키지 업데이트
sudo apt update && sudo apt upgrade -y

# 필수 도구 설치
sudo apt install -y git curl build-essential

# 시스템 정보 확인
uname -a
free -h
df -h
```

---

## 비용 절약 팁

### VM 중지하기

사용하지 않을 때 VM을 중지하면 컴퓨팅 비용이 청구되지 않습니다 (디스크 비용만 발생):

```bash
# VM 중지
gcloud compute instances stop openclaw-vm --zone=asia-northeast3-a

# VM 시작
gcloud compute instances start openclaw-vm --zone=asia-northeast3-a
```

### 고정 IP 예약 (선택)

VM을 중지/시작하면 외부 IP가 변경됩니다. 고정하려면:

```bash
# 고정 IP 예약
gcloud compute addresses create openclaw-ip --region=asia-northeast3

# 예약된 IP 확인
gcloud compute addresses describe openclaw-ip --region=asia-northeast3
```

---

## 문제 해결

### "Quota exceeded"

무료 체험에서는 리소스 제한이 있습니다:
- 더 작은 머신 타입 선택
- 다른 리전 시도

### SSH 접속 실패

```bash
# 방화벽 규칙 확인
gcloud compute firewall-rules list

# SSH 규칙이 없으면 추가
gcloud compute firewall-rules create allow-ssh \
  --allow=tcp:22 \
  --source-ranges=0.0.0.0/0
```

### "Connection refused"

VM이 시작되지 않았을 수 있습니다:

```bash
# VM 상태 확인
gcloud compute instances list

# VM 시작
gcloud compute instances start openclaw-vm --zone=asia-northeast3-a
```

---

## 다음 단계

VM이 생성되고 SSH 접속이 확인되었으면 다음 문서로 진행하세요:

→ [04. OpenClaw 설치 및 설정](openclaw-setup-guide.md)

---

*최종 업데이트: 2026-02-01*
