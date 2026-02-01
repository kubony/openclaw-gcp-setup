---
summary: "외부 스킬 디렉토리 통합 (extraDirs) - Claude Code 스킬을 OpenClaw에서 사용하기"
read_when:
  - 외부 프로젝트의 스킬을 OpenClaw에 통합할 때
  - Claude Code 스킬을 OpenClaw에서 사용하고 싶을 때
  - 여러 워크스페이스의 스킬을 공유할 때
title: "External Skills Integration (extraDirs)"
---

# 외부 스킬 디렉토리 통합 (extraDirs)

OpenClaw에서 외부 프로젝트(예: Obsidian 볼트, 다른 Claude Code 프로젝트)의 스킬을 자동으로 로드하는 방법입니다.

## 문제 상황

Claude Code 프로젝트에서 만든 커스텀 스킬들(`.claude/skills/`)이 OpenClaw 텔레그램 봇에서 자동으로 인식되지 않는 경우:

- OpenClaw는 기본적으로 자체 워크스페이스(`~/.openclaw/workspace/skills/`)와 번들 스킬만 로드
- 외부 프로젝트의 `.claude/skills/`는 별도 시스템이라 자동 인식 안 됨
- 매번 "그 스킬 참조해"라고 명시적으로 지시해야 함

## 해결 방법

### 1단계: skills.load.extraDirs 설정

`~/.openclaw/openclaw.json`에 외부 스킬 경로 추가:

```json
{
  "skills": {
    "load": {
      "extraDirs": [
        "/path/to/your/project/.claude/skills",
        "/path/to/another/project/.claude/skills"
      ]
    },
    "install": {
      "nodeManager": "pnpm"
    },
    "entries": {
      // 기존 설정...
    }
  }
}
```

**예시 (Obsidian 볼트):**

```json
{
  "skills": {
    "load": {
      "extraDirs": ["/home/user/obsidian/.claude/skills"]
    }
  }
}
```

### 2단계: 에이전트 라우팅 추가 (선택사항)

외부 프로젝트에 에이전트(`.claude/agents/*.md`)도 있다면, `~/.openclaw/workspace/AGENTS.md`에 라우팅 섹션 추가:

```markdown
## 외부 에이전트 라우팅

다음 요청 시 해당 에이전트 파일을 **먼저 읽고** 지시사항을 따르세요:

| 트리거 키워드 | 에이전트 파일 |
|-------------|-------------|
| "볼트 정리" | `/path/to/project/.claude/agents/vault-organizer.md` |
| "인물 검색" | `/path/to/project/.claude/agents/knowledge-orchestrator.md` |

**중요**: 에이전트 파일에는 단계별 워크플로우가 명시되어 있습니다. 파일을 읽지 않고 임의로 처리하지 마세요.
```

### 3단계: 게이트웨이 재시작

설정 적용을 위해 게이트웨이 재시작:

```bash
pkill -9 -f openclaw-gateway || true
nohup openclaw gateway run --bind loopback --port 18789 --force > /tmp/openclaw-gateway.log 2>&1 &
```

### 4단계: 검증

스킬 로드 확인:

```bash
openclaw skills list
```

`Source` 컬럼에서 `openclaw-extra`로 표시되면 extraDirs에서 로드된 스킬입니다.

## 스킬 우선순위

동일한 이름의 스킬이 여러 위치에 있을 경우:

1. **워크스페이스 스킬** (`<workspace>/skills/`) - 최우선
2. **관리 스킬** (`~/.openclaw/skills/`)
3. **extraDirs 스킬** - 설정 순서대로
4. **번들 스킬** - 최하위

## Claude Code 스킬 호환성

Claude Code의 `SKILL.md` 포맷은 OpenClaw와 호환됩니다:

```markdown
---
name: my-skill
description: 스킬 설명
---

# 스킬 제목

스킬 사용 지침...
```

**주의사항:**

- `metadata.openclaw.requires.bins` 게이팅은 OpenClaw에서도 동작
- Python 스크립트 경로는 절대 경로 사용 권장
- 가상환경(venv) 경로도 절대 경로로 명시

## 실제 사용 예시

### Obsidian 지식베이스 통합

```json
{
  "skills": {
    "load": {
      "extraDirs": ["/home/user/obsidian/.claude/skills"]
    }
  }
}
```

통합되는 스킬 예시:
- `ontology-engine` - 인물/프로젝트 시맨틱 검색
- `audio-transcriber` - 녹음 파일 STT 변환
- `gmail-reader/sender` - 이메일 연동
- `sheets-sync` - 구글시트 동기화

### 여러 프로젝트 통합

```json
{
  "skills": {
    "load": {
      "extraDirs": [
        "/home/user/obsidian/.claude/skills",
        "/home/user/work-project/.claude/skills",
        "~/shared-skills"
      ]
    }
  }
}
```

## 트러블슈팅

### 스킬이 로드되지 않음

1. 경로가 정확한지 확인 (`ls /path/to/skills/`)
2. 각 스킬 폴더에 `SKILL.md`가 있는지 확인
3. YAML frontmatter에 `name`과 `description` 필드가 있는지 확인
4. 게이트웨이 재시작 후 다시 확인

### 스킬이 "missing" 상태

- `metadata.openclaw.requires.bins`에 명시된 바이너리가 설치되지 않음
- 해당 CLI 도구 설치 후 다시 확인

### 에이전트가 인식되지 않음

- 에이전트(`.claude/agents/`)는 extraDirs로 로드되지 않음
- `AGENTS.md`에 명시적 라우팅 추가 필요

## 관련 문서

- [Skills](/tools/skills) - 스킬 기본 개념
- [Skills Config](/tools/skills-config) - 전체 설정 스키마
- [Creating Skills](/tools/creating-skills) - 스킬 작성 가이드
