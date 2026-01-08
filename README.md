# Claude Skills Collection

Claude Code에서 `/plugin` 명령으로 설치하여 사용할 수 있는 스킬 모음입니다.

## 스킬 목록

| 스킬 | 설명 |
|------|------|
| [security-review](skills/security-review) | OWASP Top 10 2025 기반 보안 취약점 검토 |
| [code-review](skills/code-review) | 보안, 성능, 유지보수성 관점의 코드 리뷰 |
| [api-design](skills/api-design) | RESTful API 및 GraphQL 스키마 설계 |
| [test-generator](skills/test-generator) | 단위/통합/E2E 테스트 생성 |
| [korean-docs](skills/korean-docs) | 전문적인 한국어 기술 문서 작성 |
| [git-workflow](skills/git-workflow) | Git 브랜치 전략 및 커밋 컨벤션 가이드 |

## 설치 방법

### 방법 1: Marketplace 등록 후 설치

```bash
# 1. Marketplace 등록
/plugin marketplace add roboco-io/claude-skills

# 2. 원하는 스킬 설치
/plugin install security-review@roboco-skills
/plugin install code-review@roboco-skills
/plugin install api-design@roboco-skills
/plugin install test-generator@roboco-skills
/plugin install korean-docs@roboco-skills
/plugin install git-workflow@roboco-skills
```

### 방법 2: 대화형 UI로 설치

```bash
# 플러그인 매니저 열기
/plugin

# Marketplaces 탭에서 roboco-io/claude-skills 추가
# Discover 탭에서 원하는 스킬 선택하여 설치
```

### 방법 3: 로컬 설치 (개발용)

```bash
# 레포지토리 클론
git clone https://github.com/roboco-io/claude-skills.git

# Claude Code에서 로컬 경로로 marketplace 등록
/plugin marketplace add /path/to/claude-skills
```

## 스킬 사용법

설치된 스킬은 자동으로 활성화됩니다. 관련 작업을 요청하면 Claude가 해당 스킬을 사용합니다.

**예시:**
- "이 코드 보안 검토해줘" → `security-review` 스킬 활성화
- "이 코드 리뷰해줘" → `code-review` 스킬 활성화
- "사용자 API 설계해줘" → `api-design` 스킬 활성화
- "이 함수에 대한 테스트 작성해줘" → `test-generator` 스킬 활성화

## 새 스킬 만들기

새 플러그인을 만들려면 **[플러그인 제작 가이드](docs/plugin-development-guide.md)**를 참고하세요.

### 빠른 시작

1. 스킬 폴더 및 필수 파일 생성:

```
skills/my-new-skill/
├── .claude-plugin/
│   └── plugin.json       # 플러그인 메타데이터
└── SKILL.md              # 스킬 지침
```

2. `plugin.json` 작성:

```json
{
  "name": "my-new-skill",
  "description": "스킬 설명",
  "version": "1.0.0"
}
```

3. `SKILL.md` 작성:

```markdown
---
name: my-new-skill
description: 스킬에 대한 설명. Claude가 이 스킬을 언제 사용해야 하는지 명확하게 기술합니다.
---

# 스킬 제목

Claude가 따를 지침을 여기에 작성합니다.
```

4. `marketplace.json`에 플러그인 추가:

```json
{
  "plugins": [
    {
      "name": "my-new-skill",
      "source": "./skills/my-new-skill",
      "description": "스킬 설명"
    }
  ]
}
```

자세한 내용은 [플러그인 제작 가이드](docs/plugin-development-guide.md)를 참고하세요.

## 라이선스

MIT License
