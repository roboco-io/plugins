# Workflow 플러그인 (Workflow)

프로젝트의 **의도(Intent)** 를 문서화하고 학습을 통해 진화시키는 메타 워크플로 스킬을 제공합니다. 왜 만드는지, 무엇을 만들고 무엇은 만들지 않을지를 명시적으로 합의하고, 그 결정을 프로젝트 전 과정에서 참조할 수 있게 합니다.

## 특징

- **의도 중심 설계** — `intent`가 INTENT.md로 프로젝트의 Why/What/Not을 문서화하고, 탐구에서 배운 것으로 Why/What/Not을 계속 고쳐 써서 의도를 진화시킵니다.
- **피벗·종료 판단 근거** — 의도를 고쳐 쓸 때마다 무엇을 배워서 왜 바꿨는지 커밋 메시지에 남기고, 그 이력(`git log -p INTENT.md`)을 방향 전환이나 프로젝트 종료 판단의 근거로 삼습니다.

## 수록 스킬

| 스킬 | 용도 | 트리거 예시 |
|------|------|-------------|
| [intent](skills/intent) | INTENT.md로 프로젝트 의도(Why/What/Not) 문서화 | "INTENT.md 작성 도와줘", "프로젝트 방향 잡기" |

## 사용 예시

### 예시 — 새 프로젝트 시작
`intent`로 INTENT.md를 먼저 작성하여 왜 만드는지·무엇을 만들지·무엇은 만들지 않을지를 합의합니다. 탐구 결과는 별도 로그로 쌓지 않고 Why/What/Not을 직접 고쳐 써서 반영하며, 무엇을 배워서 왜 바꿨는지는 커밋 메시지에 남깁니다. 그 이력(`git log -p INTENT.md`)이 피벗·종료 판단의 근거가 됩니다.

## 설치

```bash
/plugin marketplace add roboco-io/plugins
/plugin install workflow@roboco-plugins
```

## 관련 문서

- 루트 README: [../../README.md](../../README.md)
- 플러그인 제작 가이드: [../../docs/plugin-development-guide.md](../../docs/plugin-development-guide.md)
