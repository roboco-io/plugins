# Claude Code 플러그인 제작 가이드

이 문서는 Claude Code에서 사용할 수 있는 플러그인(스킬)을 제작하는 방법을 설명합니다.

## 목차

1. [플러그인 구조](#플러그인-구조)
2. [Marketplace 설정](#marketplace-설정)
3. [플러그인 정의](#플러그인-정의)
4. [SKILL.md 작성](#skillmd-작성)
5. [설치 및 테스트](#설치-및-테스트)
6. [문제 해결](#문제-해결)

---

## 플러그인 구조

Claude Code 플러그인은 다음과 같은 디렉토리 구조를 따릅니다:

```
my-plugins/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace 정의 (필수)
├── skills/
│   └── my-skill/
│       ├── .claude-plugin/
│       │   └── plugin.json   # 플러그인 메타데이터 (필수)
│       └── SKILL.md          # 스킬 지침 (필수)
├── README.md
└── docs/
```

---

## Marketplace 설정

### marketplace.json 스키마

`.claude-plugin/marketplace.json` 파일은 플러그인 마켓플레이스를 정의합니다.

```json
{
  "name": "my-marketplace",
  "owner": {
    "name": "Your Name or Organization"
  },
  "metadata": {
    "description": "마켓플레이스 설명"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./skills/plugin-name",
      "description": "플러그인 설명"
    }
  ]
}
```

### 필수 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `name` | string | 마켓플레이스 식별자 (kebab-case) |
| `owner` | object | 소유자 정보 (`name` 필수, `email` 선택) |
| `plugins` | array | 플러그인 목록 |

### plugins 배열 항목

| 필드 | 타입 | 설명 |
|------|------|------|
| `name` | string | 플러그인 식별자 (kebab-case) |
| `source` | string | 플러그인 경로 (상대 경로) |
| `description` | string | 플러그인 설명 (선택) |

### 주의사항

plugins 배열은 **문자열이 아닌 객체**여야 합니다:

```json
// 잘못된 예
"plugins": [
  "skills/my-skill"
]

// 올바른 예
"plugins": [
  {
    "name": "my-skill",
    "source": "./skills/my-skill"
  }
]
```

---

## 플러그인 정의

### plugin.json 스키마

각 플러그인 폴더에 `.claude-plugin/plugin.json` 파일을 생성합니다.

```json
{
  "name": "my-skill",
  "description": "플러그인 설명. 언제 활성화되는지 명시합니다.",
  "version": "1.0.0"
}
```

### 필수 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `name` | string | 플러그인 식별자 (marketplace.json의 name과 일치) |
| `description` | string | 플러그인 설명 |

### 선택 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `version` | string | 버전 (semver) |
| `author` | object | 작성자 정보 |
| `homepage` | string | 홈페이지 URL |
| `license` | string | 라이선스 (SPDX) |
| `keywords` | array | 검색 키워드 |

---

## SKILL.md 작성

### 기본 구조

```markdown
---
name: my-skill
description: 스킬 설명. Claude가 이 스킬을 언제 사용해야 하는지 명시합니다.
---

# 스킬 제목

Claude가 따를 지침을 작성합니다.

## 섹션 1
...

## 섹션 2
...
```

### YAML Frontmatter

| 필드 | 설명 |
|------|------|
| `name` | 스킬 식별자 (plugin.json과 일치) |
| `description` | 스킬 활성화 조건을 포함한 상세 설명 |

### 작성 가이드라인

1. **간결하게 유지**: Context window는 공유 자원입니다
2. **명확한 범위**: 언제 활성화되는지 description에 명시
3. **구체적인 예시**: 추상적 설명보다 코드 예제가 효과적
4. **출력 형식 정의**: Claude가 따를 응답 템플릿 제공

### 예시

```markdown
---
name: code-review
description: Perform code reviews. Use when users request code review or PR review.
---

# Code Review Skill

You are an expert code reviewer.

## Review Process

1. Read the entire changeset
2. Check for security issues
3. Evaluate code quality

## Output Format

Structure your review as:
- Summary
- Issues found
- Recommendations
```

---

## 설치 및 테스트

### GitHub에서 설치

```bash
# 1. Marketplace 등록
/plugin marketplace add owner/repo

# 2. 플러그인 설치
/plugin install plugin-name@marketplace-name
```

### 로컬에서 테스트

```bash
# 로컬 경로로 marketplace 등록
/plugin marketplace add /path/to/my-plugins

# 플러그인 설치
/plugin install my-skill@my-marketplace
```

### 설치 확인

```bash
# 설치된 플러그인 목록
/plugin list

# 대화형 UI
/plugin
```

### 설치 범위

| 범위 | 설명 |
|------|------|
| user | 모든 프로젝트에서 사용 (기본값) |
| project | 현재 프로젝트의 모든 협업자 |
| local | 현재 프로젝트에서 본인만 |

```bash
/plugin install my-skill@my-marketplace --scope project
```

---

## 문제 해결

### "Marketplace not found" 오류

**원인**: 잘못된 URL 형식 사용

```bash
# 잘못된 예
/plugin install https://github.com/owner/repo

# 올바른 예
/plugin marketplace add owner/repo
/plugin install plugin-name@marketplace-name
```

### "Invalid schema" 오류

**원인**: marketplace.json 스키마 오류

체크리스트:
- [ ] `owner` 필드가 있는가?
- [ ] `owner.name`이 있는가?
- [ ] `plugins` 배열 항목이 객체인가?
- [ ] 각 플러그인에 `name`과 `source`가 있는가?

### "origin/HEAD" 오류

**원인**: 새 레포지토리에서 origin/HEAD가 설정되지 않음

```bash
# 해결 방법
git remote set-head origin -a
```

### 플러그인이 활성화되지 않음

체크리스트:
- [ ] SKILL.md의 description이 명확한가?
- [ ] plugin.json의 name이 SKILL.md의 name과 일치하는가?
- [ ] marketplace.json의 source 경로가 올바른가?

### 검증 명령어

```bash
# 플러그인 구조 검증
/plugin validate .

# marketplace 목록 확인
/plugin marketplace list

# marketplace 업데이트
/plugin marketplace update marketplace-name
```

---

## 참고 자료

- [Claude Code 플러그인 공식 문서](https://docs.anthropic.com/en/docs/claude-code/plugins)
- [Anthropic Skills 레포지토리](https://github.com/anthropics/skills)
