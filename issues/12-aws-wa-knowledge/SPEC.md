# Issue #12: aws-wa-knowledge 스킬 스펙

> **스코프**: AWS Well-Architected 프레임워크 지식을 LLM Wiki 형태로 번들하고, 기존 WAR 리뷰 스킬이 이를 근거로 인용하도록 개선.
> **이슈**: #12

## 1. 개요

### 1.1 목적

AWS Well-Architected Framework 공식 문서(6 Pillar 본문 + WA Tool 질문 세트)를 llm-wiki 페이지 규약으로 컴파일하여 security 플러그인에 번들한다. 기존 `aws-well-architected` 오케스트레이터와 6개 `aws-wa-*` Pillar 스킬은 리뷰 수행 시 이 위키를 근거로 인용한다.

### 1.2 핵심 가치

- **근거 있는 리뷰**: 발견사항마다 공식 BP ID(예: `SEC01-BP01`)와 위키 페이지를 인용 — 체크리스트 암기가 아닌 문서 기반 판단
- **설치 즉시 사용**: 컴파일된 위키를 플러그인에 번들. 사용자는 크롤링·적재 과정 없음
- **유지보수자 갱신 모델**: 크롤→컴파일→검증 파이프라인은 이 레포의 유지보수 절차. 품질 통제 후 릴리스로 배포
- **Graceful degrade**: 위키 페이지가 없거나 누락된 항목은 기존 내장 체크리스트로 동작 — 기존 사용자 경험 파손 없음
- **확장 가능 구조**: Phase 2에서 렌즈(Serverless, SaaS 등)를 위키 디렉토리 추가만으로 수용

### 1.3 인터뷰 결정 사항

| 항목 | 결정 |
|------|------|
| 위키 콘텐츠 | 프레임워크 지식 (리뷰 이력 아님) |
| 지식 출처 | AWS 공식 문서 크롤링 |
| 배포 형태 | 플러그인 번들 (컴파일 결과물을 레포에 커밋) |
| 기존 스킬 관계 | 기존 7개 스킬 개선 (신규 병행/대체 아님) |
| 적재 범위 | 프레임워크 본문 + WA Tool 질문 세트 (렌즈는 Phase 2) |
| 갱신 주체 | 유지보수자만 (사용자 노출 update 명령 없음) |
| 위키 언어 | 영어 원문 (리포트 생성 시에만 한국어 변환) |

## 2. 아키텍처

### 2.1 신규 스킬 배치

```text
plugins/security/skills/aws-wa-knowledge/
├── SKILL.md                    # 얇은 지침: 위키 탐색·인용 규약 (~100줄)
└── wiki/
    ├── index.md                # Pillar → 질문 → 페이지 라우팅 테이블
    ├── framework.md            # 프레임워크 개요·버전·General Design Principles
    ├── operational-excellence/
    │   ├── _pillar.md          # Pillar 개요 + 설계 원칙
    │   ├── ops-01.md           # WA Tool 질문 단위 페이지 (OPS 1)
    │   └── ...
    ├── security/
    │   ├── _pillar.md
    │   ├── sec-01.md
    │   └── ...
    ├── reliability/ …
    ├── performance-efficiency/ …
    ├── cost-optimization/ …
    └── sustainability/ …
```

- 페이지 단위는 **WA Tool 질문**(약 57개). 각 질문 페이지가 소속 BP들(제목, 위험 수준, 핵심 가이던스 요약, 공식 문서 URL)을 포함.
- Pillar당 `_pillar.md`가 설계 원칙과 질문 목록을 보유.
- Phase 2 렌즈는 `wiki/lenses/{lens-name}/` 디렉토리로 추가 (이번 스코프 아님, 구조만 예약).

### 2.2 페이지 frontmatter 규약

llm-wiki 규약을 준용하되 WA 도메인 필드를 추가:

```yaml
---
title: "SEC 1. How do you securely operate your workload?"
type: policy
pillar: security
question_id: SEC-01
best_practices: [SEC01-BP01, SEC01-BP02, ...]
tags: [security, iam]
framework_version: "2024-06-27"
sources:
  - https://docs.aws.amazon.com/wellarchitected/latest/framework/...
updated: 2026-07-22
---
```

- 본문은 영어 원문 기반 컴파일 (요약·재구성 허용, 의미 왜곡 금지)
- 페이지 간 교차 참조는 `[[pillar/question-id]]` 형식

### 2.3 기존 스킬 개선

**오케스트레이터 (`aws-well-architected`)**:

- 리뷰 시작 시 `aws-wa-knowledge/wiki/index.md` 존재 확인 → 있으면 "위키 인용 모드", 없으면 기존 모드
- 통합 리포트 템플릿에 **근거(Evidence) 컬럼** 추가: 발견사항마다 BP ID + 위키 페이지 링크

**Pillar 스킬 (`aws-wa-*` 6개)**:

- SKILL.md에 "위키 인용 규약" 섹션 추가: 점검 항목 판단 시 해당 질문 페이지를 읽고 BP 기준으로 평가, 발견사항에 `근거: SEC01-BP01 ([[security/sec-01]])` 형식 인용
- 위키 접근 경로: 같은 security 플러그인 내 상대 경로 (`../aws-wa-knowledge/wiki/`)
- 해당 질문 페이지가 없으면 기존 내장 체크리스트로 판단하고 리포트에 "내장 기준" 표기

### 2.4 유지보수자 빌드 파이프라인

```text
docs/wa-wiki-build-guide.md     # LLM 주도 빌드 워크플로 문서
scripts/wa-wiki/
└── validate.mjs                # 위키 무결성 검증 (테스트에서도 재사용)
```

빌드 워크플로 (유지보수자가 Claude Code에서 수행):

1. **크롤**: AWS 공식 문서(프레임워크 본문 + Pillar 백서 + WA Tool 질문)를 WebFetch로 수집
2. **컴파일**: 질문 단위 페이지로 컴파일 (frontmatter + BP 목록 + 교차 링크)
3. **검증**: `npm test` (아래 2.5) 통과 확인
4. **커밋·릴리스**: framework_version 갱신, CHANGELOG 기록

사용자 노출 명령은 없음. 빌드 절차는 문서로만 유지.

### 2.5 테스트

`src/__tests__/wa-wiki.test.ts` 신설:

- 모든 위키 페이지의 frontmatter 스키마 검증 (필수 필드, question_id 형식, framework_version 일치)
- `[[링크]]` 무결성: 참조 대상 페이지 존재 확인
- `index.md` 커버리지: 모든 질문 페이지가 index에 등재
- BP ID 형식 검증 (`^[A-Z]+\d{2}-BP\d{2}$`)
- 6개 Pillar 디렉토리 존재 + `_pillar.md` 존재

기존 `skills.test.ts`의 500줄 제한은 SKILL.md에만 적용되므로 위키 페이지는 별도 기준(제한 없음, 단 질문당 1페이지).

## 3. 스코프 제외 (Non-Goals)

- 리뷰 결과 이력의 위키 적재 (프로젝트별 llm-wiki 영역, 필요 시 별도 이슈)
- 사용자용 위키 갱신 명령 (유지보수자 전용)
- 위키 한국어 번역
- 렌즈 적재 (Phase 2, 별도 이슈)
- qmd/LanceDB 검색 연동 (index.md 라우팅으로 충분, 필요 시 추후)

## 4. 성공 기준

1. `npm test` 통과 (신규 wa-wiki 테스트 포함)
2. 위키에 6개 Pillar 전체 질문 페이지(~57개)가 framework_version과 함께 적재됨
3. IaC 샘플에 대해 WAR 리뷰 실행 시 발견사항에 BP ID·위키 페이지 인용이 포함됨
4. `aws-wa-knowledge` 미설치(페이지 부재) 상태에서도 기존 리뷰가 동일하게 동작
5. `.claude-plugin/marketplace.json`의 security 플러그인 `skills` 배열에 `./skills/aws-wa-knowledge` 추가, README 플러그인 목록 반영
