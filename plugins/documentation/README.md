# Documentation 플러그인 (Documentation)

프로젝트 문서(스펙·ADR·회의록)를 LLM이 컴파일하는 교차참조 위키로 축적하는 문서화 워크플로를 제공합니다. 매번 원문을 다시 검색하는 RAG 방식이 아니라, 한 번 정리한 지식이 위키에 누적되어 복리로 쌓이는 모델입니다.

## 특징

- **지식 컴파일** — `llm-wiki`가 Karpathy LLM Wiki 패턴으로 프로젝트 문서를 `wiki/`에 교차참조 마크다운으로 컴파일합니다.
- **점진적 검색 강화** — qmd 설치 시 하이브리드 검색이 자동 활성되고, 미설치 시에는 INDEX.md 라우팅으로 동작합니다. LanceDB + Embedding Atlas 벡터 인덱스는 선택적입니다.
- **Obsidian 호환** — `[[wiki-link]]` 문법을 사용하므로 `wiki/`를 Obsidian vault로 바로 열 수 있습니다.

## 수록 스킬

| 스킬 | 용도 | 트리거 예시 |
|------|------|-------------|
| [llm-wiki](skills/llm-wiki) | Karpathy LLM Wiki 패턴으로 프로젝트 문서를 컴파일 위키로 유지 | "위키 초기화", "wiki ingest", "프로젝트 컨텍스트 위키", "knowledge base 구축" |

## 사용 예시

### 예시 — 프로젝트 지식을 위키로 컴파일
`llm-wiki init`으로 `wiki/` 구조를 생성하고 기존 문서(스펙·ADR·회의록)를 `wiki/raw/`에 드롭한 뒤 `llm-wiki ingest`로 컴파일합니다. qmd 설치 시 하이브리드 검색이, LanceDB 설치 시 벡터 시각화(Embedding Atlas)가 자동 활성됩니다. 이후 `query`·`lint`·`sync`·`export`로 지식을 유지·활용합니다.

## 설치

```bash
/plugin marketplace add roboco-io/plugins
/plugin install documentation@roboco-plugins
```

## 관련 문서

- 루트 README: [../../README.md](../../README.md)
- 플러그인 제작 가이드: [../../docs/plugin-development-guide.md](../../docs/plugin-development-guide.md)
