# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

GCP 인프라 자동화를 위한 Claude Code 스킬 모음입니다. 19개의 스킬로 구성되어 있으며, 슬래시 커맨드(`/gcp-*`)로 호출합니다.

## 아키텍처

```
.claude/
├── context.md          # GCP 환경 정보 (사용자별 설정)
└── skills/
    ├── gcp-billing*/   # 빌링 조회 (4개)
    ├── gcp-vm-*/       # VM 관리 (5개)
    ├── gcp-project-*/  # 프로젝트 관리 (3개)
    └── gcp-*/          # 기타 인프라 (7개)
```

### 스킬 카테고리

| 카테고리 | 스킬 | 용도 |
|----------|------|------|
| **빌링** | `gcp-billing`, `gcp-billing-accounts`, `gcp-billing-projects`, `gcp-usage` | BigQuery 기반 비용/사용량 분석 |
| **VM** | `gcp-vm-create`, `gcp-vm-control`, `gcp-vm-resize`, `gcp-vm-ssh`, `gcp-vm-init` | VM 생성부터 SSH 접속까지 |
| **프로젝트** | `gcp-project-setup`, `gcp-projects`, `gcp-project-status`, `gcp-cleanup` | 프로젝트 생성/조회/정리 |
| **서버리스** | `gcp-cloudrun`, `gcp-functions` | 서버리스 배포 |
| **인프라** | `gcp-storage`, `gcp-firewall`, `gcp-snapshot`, `gcp-iam`, `gcp-secret`, `gcp-logs`, `gcp-alerts` | 스토리지, 네트워크, 보안, 모니터링 |

## 스킬 개발

새 스킬 추가 시 `.claude/skills/gcp-[name]/SKILL.md` 생성:

```yaml
---
name: gcp-[name]
description: "스킬 설명. 트리거: 한국어 키워드들"
---
```

참조 파일이 필요하면 `references/` 하위에 작성 (예: `gcp-vm-create/references/vm-presets.md`).

## Billing Export 설정 (빌링 조회 시 필요)

빌링 스킬을 사용하려면 `.claude/context.md`에 Billing Export 정보를 설정해야 합니다:

```markdown
## Billing Export 설정

| 항목 | 값 |
|------|-----|
| Billing Export 프로젝트 | `your-billing-project` |
| 데이터셋 | `billing_export` |
| 테이블 패턴 | `gcp_billing_export_v1_*` |
| 통화 단위 | KRW 또는 USD |
```

### Billing Export 활성화 방법

1. GCP Console → Billing → Billing export
2. BigQuery export 활성화
3. 프로젝트/데이터셋 선택
