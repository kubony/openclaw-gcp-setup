# OpenClaw GCP Onboarding 요구사항 명세

> 작성일: 2026-02-01
> 상태: 확정

## 개요

비개발자/입문자가 GCP VM에 OpenClaw를 설치하고 첫 텔레그램 봇 메시지를 받을 수 있도록 안내하는 시스템 구축

## 대상 사용자

- **기술 수준**: CLI 초보자, 터미널 처음 사용
- **요구사항**: 스크린샷, 상세 설명, 단계별 체크리스트 필수

## Scope

### 포함

| 단계 | 내용 |
|------|------|
| 1 | GCP $300 무료 크레딧 활성화 안내 |
| 2 | Google Cloud SDK 설치/인증 |
| 3 | GCP VM 생성 (기존 스킬 활용) |
| 4 | SSH 접속 설정 |
| 5 | OpenClaw 설치 |
| 6 | 텔레그램 봇 연동 |
| 7 | 첫 메시지 전송 확인 |

### 제외

- 슬랙 연동 (향후 추가 가능)
- 고급 OpenClaw 설정
- Cloud Run/Functions 배포

## Deliverables

### 문서 (docs/)

| 파일 | 설명 |
|------|------|
| `01-gcp-account.md` | GCP 계정 생성 + 무료 크레딧 활성화 |
| `02-gcloud-install.md` | Google Cloud SDK 설치 + 인증 |
| `03-vm-create.md` | VM 생성 (스크린샷 포함) |
| `04-openclaw-install.md` | OpenClaw 설치 + 텔레그램 봇 연동 |

### 에이전트 (.claude/agents/)

| 파일 | 설명 |
|------|------|
| `openclaw-onboarding.md` | 단계별 안내 서브에이전트. 기존 `/gcp-*` 스킬들과 연동 |

### README 업데이트

- 온보딩 문서 링크 추가
- 에이전트 사용법 안내

## 기술적 결정

| 항목 | 결정 | 이유 |
|------|------|------|
| 에이전트 형태 | Claude Code 서브에이전트 | 기존 스킬들과 통합, 대화형 안내 |
| 문서 구조 | 단계별 분리 (01~04) | 진행 상황 추적 용이 |
| 메시지 연동 | 텔레그램 | 보편적, 설정 간편 |
| VM 사양 | e2-standard-4 권장 | 비용/성능 균형 |

## Success Criteria

1. ✅ 문서만 따라해도 GCP VM 생성 가능
2. ✅ 에이전트가 단계별 진행 상황 체크
3. ✅ 최종: 텔레그램에서 봇 메시지 수신 확인

## 참조

- 기존 문서: `docs/macmini-grade-vm-setup.md`
- 기존 스킬: `/gcp-vm-create`, `/gcp-vm-ssh`, `/gcp-project-setup`
