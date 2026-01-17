# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Claude Code 플러그인 마켓플레이스 레포지토리. Skills, Commands, Agents, Hooks를 제공합니다.

- **GitHub**: `roboco-io/plugins`
- **Marketplace 이름**: `roboco-plugins`

## Architecture

```
.claude-plugin/
└── marketplace.json          # Marketplace 정의 (필수)

skills/{skill-name}/
├── .claude-plugin/
│   └── plugin.json           # 플러그인 메타데이터
├── SKILL.md                  # 스킬 지침 (YAML frontmatter + Markdown)
└── references/               # Progressive disclosure용 참조 파일 (선택)
    ├── patterns/             # 언어별 상세 패턴
    └── checklists/           # 체크리스트
```

## Plugin Development

### 새 플러그인 추가 시 필수 작업

1. `skills/{name}/.claude-plugin/plugin.json` 생성
2. `skills/{name}/SKILL.md` 생성 (YAML frontmatter 포함)
3. `.claude-plugin/marketplace.json`의 `plugins` 배열에 추가

### marketplace.json 스키마

```json
{
  "name": "roboco-plugins",
  "owner": { "name": "roboco-io" },
  "plugins": [
    {
      "name": "skill-name",
      "source": "./skills/skill-name",
      "description": "설명"
    }
  ]
}
```

**주의**: `plugins` 배열 항목은 문자열이 아닌 객체여야 함.

### SKILL.md 구조

```markdown
---
name: skill-name
description: 스킬 설명. Claude가 언제 이 스킬을 사용해야 하는지 명시.
---

# 스킬 제목

[지침 내용]
```

### Progressive Disclosure

SKILL.md는 500줄 이하로 유지하고, 상세 내용은 `references/` 디렉토리에 분리:
- SKILL.md에서 `[file.md](references/file.md)` 형식으로 참조
- Claude가 필요할 때만 참조 파일을 로드함

## Testing

```bash
# 로컬에서 marketplace 등록
/plugin marketplace add /path/to/plugins

# 플러그인 설치
/plugin install skill-name@roboco-plugins

# 검증
/plugin validate .
```

## Troubleshooting

- **origin/HEAD 오류**: `git remote set-head origin -a` 실행
- **Invalid schema 오류**: marketplace.json에 `owner.name` 필드 확인
- **플러그인 미활성화**: SKILL.md의 `name`과 plugin.json의 `name` 일치 여부 확인
