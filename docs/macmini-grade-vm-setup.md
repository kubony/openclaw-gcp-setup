# 맥미니급 GCP VM 설정 가이드

이 문서는 맥미니(M4)와 유사한 성능의 GCP VM을 설정하는 과정을 정리한 것입니다.

## 1. 맥미니 vs GCP VM 사양 비교

### 맥미니 사양 기준

| 모델 | CPU | RAM | 특징 |
|------|-----|-----|------|
| **M4 맥미니** | 10코어 (4P+6E) | 16-24GB | 일반 사용 |
| **M4 Pro 맥미니** | 12-14코어 | 24-48GB | 전문가용 |

### GCP VM 추천

| VM 타입 | vCPU | RAM | 월 비용 | 특징 |
|---------|------|-----|---------|------|
| **e2-standard-8** | 8 | 32GB | ~$150 | 가성비 추천 |
| **n2-standard-8** | 8 | 32GB | ~$200 | Intel Ice Lake, 높은 클럭 |
| **n2d-standard-8** | 8 | 32GB | ~$170 | AMD EPYC, 비용 대비 성능 우수 |
| **c2-standard-8** | 8 | 32GB | ~$230 | 3.1GHz 지속, 컴파일/빌드에 최적 |

### 용도별 추천

| 용도 | 추천 VM | 이유 |
|------|---------|------|
| 개발 서버 | e2-standard-8 | 가성비, 충분한 성능 |
| CI/CD 빌드 | c2-standard-8 | 높은 클럭, 빌드 속도 |
| 웹 서비스 | n2-standard-8 | 균형잡힌 성능 |
| 비용 절감 | n2d-standard-8 | AMD 기반 저렴 |

---

## 2. 비용 분석

### 월 비용 (asia-northeast3, 서울 기준)

| VM 타입 | 월 비용 (24시간) | Spot VM (70% 할인) |
|---------|------------------|-------------------|
| e2-standard-8 | ~$150 | ~$45 |
| n2-standard-8 | ~$200 | ~$60 |
| n2d-standard-8 | ~$170 | ~$50 |
| c2-standard-8 | ~$230 | ~$70 |

### 추가 비용

| 항목 | 비용 |
|------|------|
| SSD 100GB | ~$17/월 |
| SSD 256GB | ~$40/월 |
| 네트워크 (아웃바운드) | ~$0.12/GB |
| 고정 IP | ~$7/월 |

### 총 예상 (n2-standard-8 + 100GB SSD)

| 결제 방식 | 월 비용 | 1년 총비용 | 비고 |
|-----------|---------|------------|------|
| 온디맨드 | ~$220 | ~$2,640 | 언제든 해지 가능 |
| 1년 약정 (CUD) | ~$140 | ~$1,680 | 37% 할인 |
| 3년 약정 (CUD) | ~$100 | ~$1,200 | 57% 할인 |
| Spot VM | ~$70 | ~$840 | 중단 가능성 있음 |

---

## 3. 실제 진행 내역 (2026-01-31)

### 3.1 기존 VM 상태 확인

```bash
gcloud compute instances describe knowledge-hub-vm \
  --zone=asia-northeast3-a \
  --project=seo-knowledge-hub
```

**기존 사양:**
| 항목 | 값 | 문제 |
|------|-----|------|
| 머신 타입 | e2-small | 2 vCPU, 2GB RAM |
| 디스크 | 20GB | 작음 |
| 스왑 사용량 | ~1GB | RAM 부족으로 버벅임 |

### 3.2 VM 리사이즈 절차

#### Step 1: VM 중지

```bash
gcloud compute instances stop knowledge-hub-vm \
  --zone=asia-northeast3-a \
  --project=seo-knowledge-hub
```

#### Step 2: 머신 타입 변경

```bash
gcloud compute instances set-machine-type knowledge-hub-vm \
  --machine-type=n2-standard-8 \
  --zone=asia-northeast3-a \
  --project=seo-knowledge-hub
```

#### Step 3: 디스크 확장 (20GB → 100GB)

```bash
gcloud compute disks resize knowledge-hub-vm \
  --size=100GB \
  --zone=asia-northeast3-a \
  --project=seo-knowledge-hub
```

#### Step 4: VM 시작

```bash
gcloud compute instances start knowledge-hub-vm \
  --zone=asia-northeast3-a \
  --project=seo-knowledge-hub
```

#### Step 5: 디스크 파티션 확장 (VM 내부)

```bash
# SSH 접속 후
sudo growpart /dev/sda 1
sudo resize2fs /dev/sda1
```

### 3.3 SSH 설정 업데이트

VM 재시작으로 외부 IP가 변경됨.

#### 새 IP 확인

```bash
gcloud compute instances describe knowledge-hub-vm \
  --zone=asia-northeast3-a \
  --project=seo-knowledge-hub \
  --format="get(networkInterfaces[0].accessConfigs[0].natIP)"
```

#### ~/.ssh/config 업데이트

```
Host knowledge-hub
  HostName 34.47.68.148  # 새 IP
  User your-username
  IdentityFile ~/.ssh/google_compute_engine
```

#### 호스트 키 충돌 해결

IP 변경으로 인한 호스트 키 충돌 발생 시:

```bash
ssh-keygen -R 34.158.211.172  # 이전 IP 키 제거
ssh-keygen -R 34.47.68.148    # 새 IP 키 제거 (필요시)
```

### 3.4 최종 결과

| 항목 | 변경 전 | 변경 후 |
|------|---------|---------|
| 머신 타입 | e2-small (2 vCPU, 2GB) | **n2-standard-8** (8 vCPU, 32GB) |
| 디스크 | 20GB | **100GB** |
| 외부 IP | 34.158.211.172 | **34.47.68.148** |
| 스왑 사용량 | ~1GB | ~0 |

---

## 4. 유용한 명령어 모음

### VM 상태 확인

```bash
# VM 목록
gcloud compute instances list --project=seo-knowledge-hub

# VM 상세 정보
gcloud compute instances describe knowledge-hub-vm \
  --zone=asia-northeast3-a \
  --project=seo-knowledge-hub

# SSH 접속
gcloud compute ssh knowledge-hub-vm \
  --zone=asia-northeast3-a \
  --project=seo-knowledge-hub
```

### 리소스 사용량 확인 (VM 내부)

```bash
# 메모리/스왑 확인
free -h

# 디스크 사용량
df -h

# CPU 정보
nproc
lscpu | grep "CPU(s)"

# 실행 중 프로세스
htop  # 또는 top
```

### 비용 확인

```bash
# 프로젝트 비용 조회 (BigQuery)
bq query --project_id=patent-481206 --use_legacy_sql=false "
SELECT
  ROUND(SUM(cost)) AS cost_krw
FROM \`patent-481206.billing_export.gcp_billing_export_v1_*\`
WHERE project.id = 'seo-knowledge-hub'
  AND DATE(usage_start_time) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
"
```

---

## 5. 주의사항

1. **IP 변경**: VM을 중지/시작하면 외부 IP가 변경됨 (고정 IP 필요시 별도 예약)
2. **다운타임**: 리사이즈 시 5-10분 VM 중지 필요
3. **디스크 확장**: 축소는 불가, 확장만 가능
4. **요금**: 머신 타입 변경 즉시 새 요금 적용
5. **파티션 확장**: 디스크 확장 후 OS 내부에서 파티션/파일시스템 확장 필요

---

## 6. 관련 스킬

| 스킬 | 용도 |
|------|------|
| `/gcp-vm-create` | 새 VM 생성 |
| `/gcp-vm-resize` | VM 머신 타입 변경 |
| `/gcp-vm-control` | VM 시작/중지/재시작 |
| `/gcp-vm-ssh` | SSH 접속 및 터널링 |
| `/gcp-billing` | 비용 조회 |
