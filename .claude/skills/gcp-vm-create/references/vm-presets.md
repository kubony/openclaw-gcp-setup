# VM Presets

용도별 추천 사양.

## 머신 타입 추천

| 용도 | 머신 타입 | vCPU | RAM | 디스크 | 월 예상 비용 |
|------|-----------|------|-----|--------|--------------|
| 경량 (봇, 스케줄러) | e2-micro | 0.25 | 1GB | 10GB | ~$7 |
| 웹/API 서버 | e2-small | 0.5 | 2GB | 20GB | ~$13 |
| 개발 환경 | e2-medium | 1 | 4GB | 30GB | ~$27 |
| 프로덕션 웹 | e2-standard-2 | 2 | 8GB | 50GB | ~$50 |
| 데이터베이스 | e2-standard-4 | 4 | 16GB | 100GB | ~$100 |
| 고성능 | n2-standard-4 | 4 | 16GB | 100GB | ~$140 |

## 리전별 특성

| 리전 | 존 | 특성 |
|------|-----|------|
| asia-northeast3 | a, b, c | 서울, 한국 사용자 최적 |
| asia-northeast1 | a, b, c | 도쿄, 일본 사용자 |
| us-central1 | a, b, c, f | 아이오와, 가장 저렴 |
| us-west1 | a, b, c | 오레곤, 미국 서부 |

## OS 이미지

| OS | image-family | image-project |
|----|--------------|---------------|
| Ubuntu 22.04 LTS | ubuntu-2204-lts | ubuntu-os-cloud |
| Ubuntu 24.04 LTS | ubuntu-2404-lts-amd64 | ubuntu-os-cloud |
| Debian 12 | debian-12 | debian-cloud |
| CentOS Stream 9 | centos-stream-9 | centos-cloud |
| Rocky Linux 9 | rocky-linux-9 | rocky-linux-cloud |

## 방화벽 포트

| 용도 | 포트 | 태그 |
|------|------|------|
| HTTP | 80 | http-server |
| HTTPS | 443 | https-server |
| SSH | 22 | (기본 허용) |
| Node.js 개발 | 3000 | nodejs-server |
| PostgreSQL | 5432 | postgres-server |
| MySQL | 3306 | mysql-server |
