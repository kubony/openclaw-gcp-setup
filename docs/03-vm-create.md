# 3. GCP VM 생성

이 문서는 OpenClaw를 실행할 GCP VM(가상 머신)을 생성하는 방법을 안내합니다.

---

## 용어 설명

| 용어 | 설명 |
|------|------|
| **VM** | Virtual Machine. 클라우드에서 빌려 쓰는 컴퓨터 |
| **인스턴스** | VM과 같은 말. GCP에서는 "VM 인스턴스"라고 부름 |
| **vCPU** | 가상 CPU. 숫자가 클수록 빠름 |
| **SSH** | Secure Shell. 원격 컴퓨터에 접속하는 방법 |
| **리전/존** | 데이터센터 위치. 서울은 `asia-northeast3` |

---

## 권장 사양

OpenClaw 실행을 위한 VM 사양:

| 항목 | 최소 | 권장 | 설명 |
|------|------|------|------|
| vCPU | 2 | 4 | CPU 개수. 많을수록 빠름 |
| RAM | 4GB | 8GB | 메모리. 많을수록 여유로움 |
| 디스크 | 20GB | 50GB | 저장 공간 |
| OS | Ubuntu 22.04 | Ubuntu 24.04 | 운영체제 |

---

## 방법 1: GCP Console에서 생성 (초보자 권장)

마우스 클릭으로 VM을 만드는 방법입니다.

### Step 1: Compute Engine 페이지 이동

1. [GCP Console](https://console.cloud.google.com) 접속

2. 좌측 메뉴에서 **Compute Engine** → **VM 인스턴스** 클릭
   ```
   ┌──────────────────┐
   │ ☰ Google Cloud   │
   ├──────────────────┤
   │ > Compute Engine │ ← 클릭
   │   > VM 인스턴스  │ ← 그 다음 클릭
   │   > 디스크       │
   └──────────────────┘
   ```

3. 처음이면 "Compute Engine API 활성화" 버튼이 뜸 → 클릭하고 1-2분 대기

### Step 2: 인스턴스 만들기

1. 상단의 **"+ 인스턴스 만들기"** 버튼 클릭

2. **기본 설정**:
   ```
   이름: [openclaw-vm        ]  ← 원하는 이름 (영문, 숫자, 하이픈만)
   리전: [asia-northeast3 (서울) ▼]  ← 서울 선택
   영역: [asia-northeast3-a ▼]
   ```

3. **머신 구성**:
   ```
   시리즈: [E2 ▼]  ← 가성비 좋음
   머신 유형: [e2-medium ▼]  ← 테스트용 (2 vCPU, 4GB)
             또는 [e2-standard-4 ▼]  ← 권장 (4 vCPU, 16GB)
   ```

   | 머신 유형 | vCPU | RAM | 월 비용 | 추천 |
   |-----------|------|-----|---------|------|
   | e2-small | 2 | 2GB | ~$15 | 테스트만 |
   | e2-medium | 2 | 4GB | ~$25 | 가벼운 사용 |
   | e2-standard-4 | 4 | 16GB | ~$100 | **권장** |

4. **부팅 디스크** → **"변경"** 버튼 클릭:
   ```
   운영체제: [Ubuntu ▼]
   버전: [Ubuntu 24.04 LTS x86/64 ▼]
   부팅 디스크 유형: [SSD 영구 디스크 ▼]  ← 기본 HDD보다 빠름
   크기(GB): [50]
   ```
   → **"선택"** 클릭

5. 맨 아래 **"만들기"** 버튼 클릭

6. 1-2분 대기 → 초록색 체크 표시가 나오면 완료!

---

## 방법 2: gcloud CLI로 생성

터미널 명령어로 VM을 만드는 방법입니다. 더 빠르지만 명령어를 알아야 합니다.

### Step 1: API 활성화

```bash
gcloud services enable compute.googleapis.com
```

### Step 2: VM 생성

```bash
gcloud compute instances create openclaw-vm \
  --zone=asia-northeast3-a \
  --machine-type=e2-standard-4 \
  --image-family=ubuntu-2404-lts-amd64 \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-ssd \
  --tags=http-server,https-server
```

> **명령어 설명**:
> - `openclaw-vm`: VM 이름 (원하는 대로 변경 가능)
> - `--zone`: 서울 데이터센터
> - `--machine-type`: VM 사양 (e2-small, e2-medium 등으로 변경 가능)
> - `--image-family`: Ubuntu 24.04 LTS
> - `--boot-disk-size`: 디스크 크기
> - `--boot-disk-type`: SSD 사용

**정상 출력 예시**:
```
Created [https://www.googleapis.com/compute/v1/projects/my-project/zones/asia-northeast3-a/instances/openclaw-vm].
NAME          ZONE               MACHINE_TYPE    INTERNAL_IP  EXTERNAL_IP    STATUS
openclaw-vm   asia-northeast3-a  e2-standard-4   10.178.0.2   34.64.xxx.xxx  RUNNING
```

---

## SSH 접속하기

VM에 접속해서 명령어를 실행하려면 SSH를 사용합니다.

### 방법 1: gcloud 명령어 (가장 쉬움)

```bash
gcloud compute ssh openclaw-vm --zone=asia-northeast3-a
```

**처음 접속할 때 일어나는 일**:

1. SSH 키 생성 여부 물음:
   ```
   WARNING: The private key file for gcloud compute ... does not exist.
   Generating public/private rsa key pair.
   Enter passphrase (empty for no passphrase):
   ```
   → **그냥 엔터** 누르면 됩니다 (비밀번호 없이)

2. 다시 한번 물음:
   ```
   Enter same passphrase again:
   ```
   → **다시 엔터**

3. 접속 완료:
   ```
   Welcome to Ubuntu 24.04 LTS
   username@openclaw-vm:~$
   ```

> **username은 자동으로 정해집니다**
>
> GCP는 로컬 컴퓨터의 사용자 이름을 기반으로 VM 사용자를 자동 생성합니다.
> 예: 내 맥북 사용자가 `john`이면 VM에서도 `john`

### 방법 2: GCP Console에서 SSH

1. [VM 인스턴스 페이지](https://console.cloud.google.com/compute/instances) 접속
2. VM 목록에서 openclaw-vm 행 찾기
3. 우측 **"SSH"** 버튼 클릭
4. 새 브라우저 창에서 터미널 열림

### 방법 3: 일반 SSH 클라이언트

외부 IP 확인:
```bash
gcloud compute instances describe openclaw-vm \
  --zone=asia-northeast3-a \
  --format='get(networkInterfaces[0].accessConfigs[0].natIP)'
```

출력 예시: `34.64.123.45`

접속:
```bash
ssh -i ~/.ssh/google_compute_engine $(whoami)@34.64.123.45
```

> **$(whoami)가 뭔가요?**
>
> 현재 로그인한 사용자 이름을 자동으로 넣어줍니다.
> 직접 입력해도 됩니다: `ssh -i ~/.ssh/google_compute_engine john@34.64.123.45`

### ~/.ssh/config 설정 (편리한 접속)

매번 긴 명령어 치기 귀찮으면:

```bash
# 먼저 외부 IP 확인
gcloud compute instances describe openclaw-vm \
  --zone=asia-northeast3-a \
  --format='get(networkInterfaces[0].accessConfigs[0].natIP)'
```

`~/.ssh/config` 파일에 추가:
```
Host openclaw
  HostName 34.64.123.45
  User 여기에_본인_사용자명
  IdentityFile ~/.ssh/google_compute_engine
```

이제 간단하게:
```bash
ssh openclaw
```

---

## VM 초기 설정

SSH 접속 후 기본 설정을 합니다:

```bash
# 패키지 목록 업데이트
sudo apt update

# 패키지 업그레이드
sudo apt upgrade -y

# 필수 도구 설치
sudo apt install -y git curl build-essential

# 시스템 정보 확인
echo "=== CPU ===" && nproc
echo "=== 메모리 ===" && free -h
echo "=== 디스크 ===" && df -h
```

**정상 출력 예시**:
```
=== CPU ===
4
=== 메모리 ===
              total        used        free
Mem:           15Gi       1.2Gi        13Gi
=== 디스크 ===
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        49G  2.1G   45G   5% /
```

---

## 비용 절약 팁

### VM 중지하기

사용 안 할 때 VM을 중지하면 **컴퓨팅 비용이 0원**입니다 (디스크 비용만 ~$8/월):

```bash
# VM 중지 (컴퓨팅 비용 절약)
gcloud compute instances stop openclaw-vm --zone=asia-northeast3-a

# VM 시작
gcloud compute instances start openclaw-vm --zone=asia-northeast3-a
```

> **주의**: VM을 중지/시작하면 외부 IP가 바뀝니다!
>
> 고정 IP가 필요하면 아래 "고정 IP 예약" 참고

### 고정 IP 예약 (선택)

```bash
# 고정 IP 예약 (월 ~$7 추가)
gcloud compute addresses create openclaw-ip --region=asia-northeast3

# 예약된 IP 확인
gcloud compute addresses describe openclaw-ip --region=asia-northeast3 --format='get(address)'

# VM에 고정 IP 연결 (VM 중지 후)
gcloud compute instances delete-access-config openclaw-vm \
  --zone=asia-northeast3-a --access-config-name="External NAT"

gcloud compute instances add-access-config openclaw-vm \
  --zone=asia-northeast3-a --address=예약된_IP주소
```

---

## 문제 해결

### "Quota exceeded"

**원인**: 무료 체험 계정의 리소스 제한 초과

**해결**:
- 더 작은 머신 타입 선택 (e2-small)
- 다른 리전 시도 (us-central1 등)

### SSH 접속 실패: "Connection refused"

**원인 1**: VM이 실행 중이 아님
```bash
# VM 상태 확인
gcloud compute instances list

# 상태가 TERMINATED면 시작
gcloud compute instances start openclaw-vm --zone=asia-northeast3-a
```

**원인 2**: 방화벽에서 SSH 차단
```bash
# 방화벽 규칙 확인
gcloud compute firewall-rules list

# SSH 규칙 추가
gcloud compute firewall-rules create allow-ssh \
  --allow=tcp:22 \
  --source-ranges=0.0.0.0/0
```

### SSH 접속 실패: "Permission denied"

**원인**: SSH 키 문제

**해결**:
```bash
# SSH 키 재생성
gcloud compute config-ssh

# 다시 접속 시도
gcloud compute ssh openclaw-vm --zone=asia-northeast3-a
```

### "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"

**원인**: VM을 삭제했다가 같은 이름으로 다시 만듦 (또는 IP 변경)

**해결**:
```bash
# 기존 호스트 키 삭제
ssh-keygen -R 이전_IP주소

# 다시 접속
gcloud compute ssh openclaw-vm --zone=asia-northeast3-a
```

---

## 체크리스트

다음 단계로 넘어가기 전에 확인하세요:

- [ ] VM이 RUNNING 상태 (`gcloud compute instances list`로 확인)
- [ ] SSH 접속 성공 (`gcloud compute ssh openclaw-vm --zone=asia-northeast3-a`)
- [ ] VM 내에서 `sudo apt update` 실행됨
- [ ] VM 내에서 `git --version` 출력됨

---

## 다음 단계

VM이 생성되고 SSH 접속이 확인되었으면 다음 문서로 진행하세요:

→ [04. OpenClaw 설치 및 설정](openclaw-setup-guide.md)

---

*최종 업데이트: 2026-02-01*
