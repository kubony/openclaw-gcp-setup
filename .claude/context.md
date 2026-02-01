# GCP Infrastructure Context

사용자별 GCP 환경 정보를 이 파일에 설정합니다.

## 현재 환경

- **기본 계정**: your-account@example.com
- **조직**: (선택) your-org.com
- **기본 결제 계정**: YOUR-BILLING-ACCOUNT

---

## Billing Export 설정

빌링 스킬(`/gcp-billing` 등)을 사용하려면 아래 정보를 설정하세요.

| 항목 | 값 |
|------|-----|
| Billing Export 프로젝트 | `your-billing-project` |
| 데이터셋 | `billing_export` |
| 테이블 패턴 | `gcp_billing_export_v1_*` |
| **통화 단위** | **KRW** 또는 **USD** |

### Billing Export 활성화 방법

1. [GCP Console](https://console.cloud.google.com/billing) → Billing → Billing export
2. BigQuery export → "편집" 클릭
3. 프로젝트와 데이터셋 선택
4. "저장" 클릭

---

## 프로젝트 목록

자주 사용하는 프로젝트를 기록하세요.

### example-project

| 항목 | 값 |
|------|-----|
| 용도 | 예: 개발 서버 |
| 결제 계정 | YOUR-BILLING-ACCOUNT |
| 생성일 | 2025-01-01 |

#### VM: example-vm

| 항목 | 값 |
|------|-----|
| 존 | asia-northeast3-a (서울) |
| 머신 타입 | n2-standard-8 |
| OS | Ubuntu 22.04 LTS |
| 디스크 | 100GB SSD |
| 내부 IP | 10.x.x.x |
| 외부 IP | 34.x.x.x |

---

## 자주 쓰는 명령어

```bash
# 프로젝트 전환
gcloud config set project PROJECT_ID

# VM SSH 접속
gcloud compute ssh VM_NAME --zone=ZONE

# VM 상태 확인
gcloud compute instances list --project=PROJECT_ID
```

---

## 자동화 도구 참조

### 빌링 모니터링

| 커맨드 | 용도 |
|--------|------|
| `/gcp-billing` | 비용 분석 (프로젝트/서비스별, 30일 추이) |
| `/gcp-billing-accounts` | 결제 계정 목록 조회 |
| `/gcp-billing-projects` | 계정별 연결 프로젝트 조회 |
| `/gcp-usage` | API 사용량 조회 (SKU별, 실시간 로그) |

### 리소스 관리

| 커맨드 | 용도 |
|--------|------|
| `/gcp-projects` | 프로젝트 목록 조회 |
| `/gcp-project-status` | 프로젝트 리소스 상태 종합 조회 |
| `/gcp-cleanup` | 미사용 리소스 탐지 (읽기 전용) |

### VM 자동화

| 스킬/커맨드 | 용도 |
|------------|------|
| `gcp-project-setup` | 새 프로젝트 생성 및 설정 |
| `gcp-vm-create` | VM 생성 (용도별 사양 추천) |
| `gcp-vm-init` | VM 초기 설정 (보안/최적화) |

---

## 삭제된 프로젝트

| PROJECT_ID | 삭제일 | 사유 |
|------------|--------|------|
| - | - | - |
