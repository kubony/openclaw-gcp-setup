# OpenClaw GCP Setup

GCP 인프라 자동화를 위한 Claude Code 스킬 모음입니다.

## 스킬 목록

### 빌링 및 비용 관리
| 스킬 | 설명 |
|------|------|
| `/gcp-billing` | 프로젝트/서비스별 비용 분석 (BigQuery 기반) |
| `/gcp-billing-accounts` | 결제 계정 목록 조회 |
| `/gcp-billing-projects` | 결제 계정별 연결 프로젝트 조회 |
| `/gcp-usage` | API 사용량 조회 |

### VM 관리
| 스킬 | 설명 |
|------|------|
| `/gcp-vm-create` | VM 생성 마법사 (용도별 사양 추천) |
| `/gcp-vm-control` | VM 시작/중지/재시작 |
| `/gcp-vm-resize` | VM 머신 타입 변경 (스케일업/다운) |
| `/gcp-vm-ssh` | SSH 접속 및 터널링 설정 |
| `/gcp-vm-init` | VM 초기 설정 (보안/최적화) |

### 프로젝트 관리
| 스킬 | 설명 |
|------|------|
| `/gcp-project-setup` | 프로젝트 생성 → 결제 연결 → API 활성화 |
| `/gcp-projects` | 프로젝트 목록 조회 |
| `/gcp-project-status` | 프로젝트 리소스 상태 종합 조회 |
| `/gcp-cleanup` | 미사용 리소스 탐지 (읽기 전용) |

### 서버리스
| 스킬 | 설명 |
|------|------|
| `/gcp-cloudrun` | Cloud Run 서비스 배포/관리 |
| `/gcp-functions` | Cloud Functions 배포/관리 |

### 인프라 관리
| 스킬 | 설명 |
|------|------|
| `/gcp-storage` | Cloud Storage 버킷 관리 |
| `/gcp-firewall` | 방화벽 규칙 관리 |
| `/gcp-snapshot` | 디스크 스냅샷 생성/복원 |
| `/gcp-iam` | IAM 서비스 계정 및 권한 관리 |
| `/gcp-secret` | Secret Manager 조회/관리 |
| `/gcp-logs` | Cloud Logging 로그 조회/필터링 |
| `/gcp-alerts` | 알림 정책 설정 (비용/에러/성능) |

## 설치

### 방법 1: 프로젝트에 복사

```bash
# 레포 클론
git clone https://github.com/your-username/openclaw-gcp-setup.git

# .claude 폴더를 프로젝트에 복사
cp -r openclaw-gcp-setup/.claude /path/to/your-project/
```

### 방법 2: 전역 설치

```bash
# 전역 스킬로 설치 (모든 프로젝트에서 사용 가능)
cp -r openclaw-gcp-setup/.claude/skills/* ~/.claude/skills/
```

## 사전 요구사항

- [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) 설치
- `gcloud auth login` 으로 인증 완료
- BigQuery API 활성화 (빌링 조회 시)

## 문서

- [맥미니급 VM 설정 가이드](docs/macmini-grade-vm-setup.md)

## 라이선스

MIT
