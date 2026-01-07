# Issue #1: OWASP Top 10 2025 보안 리뷰 스킬 작업 계획

## 개요

OWASP Top 10 2025 기반의 `security-review` 스킬을 개발하여 애플리케이션 코드의 보안 취약점을 자동으로 검토합니다.

## 참고 자료

- [OWASP Top 10 2025 Introduction](https://owasp.org/Top10/2025/0x00_2025-Introduction/)
- [OWASP Top 10 2025 Next Steps](https://owasp.org/Top10/2025/X01_2025-Next_Steps/)
- [Jit.io OWASP Guide](https://www.jit.io/resources/security-standards/the-in-depth-guide-to-owasps-top-10-vulnerabilities)
- [Fastly OWASP 2025 Changes](https://www.fastly.com/blog/new-2025-owasp-top-10-list-what-changed-what-you-need-to-know)
- [Orca Security OWASP 2025 Changes](https://orca.security/resources/blog/owasp-top-10-2025-key-changes/)
- [Snyk OWASP SAST Guide](https://snyk.io/articles/owasp-top-10-code-vulnerabilities-and-how-sast-can-stop-them/)

---

## 작업 단계

### Phase 1: 기반 구조 설계

#### 1.1 스킬 구조 정의
```
skills/security-review/
├── SKILL.md              # 메인 스킬 정의
└── references/
    ├── patterns/
    │   ├── javascript.md # JS/TS 취약점 패턴
    │   ├── python.md     # Python 취약점 패턴
    │   ├── java.md       # Java 취약점 패턴
    │   └── go.md         # Go 취약점 패턴
    └── checklists/
        └── owasp-2025.md # OWASP 체크리스트
```

#### 1.2 리뷰 출력 형식 설계
```markdown
## 🔒 Security Review Report

### Summary
- Critical: X issues
- High: X issues
- Medium: X issues
- Low: X issues

### Findings

#### [CRITICAL] A05: SQL Injection
- **Location**: `src/api/users.js:42`
- **Code**: `db.query("SELECT * FROM users WHERE id = " + userId)`
- **Risk**: 공격자가 임의의 SQL 쿼리를 실행할 수 있음
- **Fix**: Prepared statement 사용
```javascript
db.query("SELECT * FROM users WHERE id = ?", [userId])
```

---

### Phase 2: OWASP Top 10 2025 상세 분석

각 취약점 카테고리별 분석 및 탐지 패턴 정의:

| 코드 | 취약점 | 심각도 | 탐지 복잡도 |
|------|--------|--------|-------------|
| A01 | Broken Access Control | Critical | Medium |
| A02 | Security Misconfiguration | High | Low |
| A03 | Software Supply Chain Failures | Critical | Medium |
| A04 | Cryptographic Failures | Critical | Low |
| A05 | Injection | Critical | Low |
| A06 | Insecure Design | High | High |
| A07 | Authentication Failures | Critical | Medium |
| A08 | Software/Data Integrity Failures | High | Medium |
| A09 | Security Logging Failures | Medium | Low |
| A10 | Mishandling Exceptional Conditions | Medium | Medium |

---

### Phase 3: 언어별 취약점 패턴 정의

#### 3.1 JavaScript/TypeScript

| OWASP | 취약 패턴 | 안전한 패턴 |
|-------|-----------|-------------|
| A01 | `req.params.id` 직접 사용 | `req.user.id` 검증 후 사용 |
| A02 | `cors({origin: '*'})` | `cors({origin: allowedOrigins})` |
| A04 | `crypto.createHash('MD5')` | `crypto.createHash('SHA-256')` |
| A05 | `db.query("... " + input)` | `db.query("... ?", [input])` |
| A05 | `eval(userInput)` | JSON.parse 또는 별도 파싱 |
| A05 | `innerHTML = userInput` | `textContent = userInput` |
| A07 | 하드코딩된 비밀번호 | 환경 변수 또는 시크릿 매니저 |
| A08 | `JSON.parse` → `eval` | 화이트리스트 검증 |
| A09 | `console.log` only | winston/pino 구조화 로깅 |
| A10 | `catch(e) { res.send(e.stack) }` | 일반 에러 메시지 반환 |

#### 3.2 Python

| OWASP | 취약 패턴 | 안전한 패턴 |
|-------|-----------|-------------|
| A01 | `Object.get(id=request.GET['id'])` | 소유권 검증 추가 |
| A02 | `app.run(debug=True)` | `app.run(debug=False)` |
| A04 | `hashlib.md5(password)` | `bcrypt.hashpw()` |
| A05 | `cursor.execute("..." + input)` | `cursor.execute("...?", (input,))` |
| A05 | `os.system(user_cmd)` | `subprocess.run([...], shell=False)` |
| A07 | `if password == plain_input` | `bcrypt.checkpw()` |
| A08 | `pickle.loads(userData)` | `json.loads()` + 검증 |
| A08 | `yaml.load(data)` | `yaml.safe_load(data)` |
| A09 | `print(error)` | `logging.error()` |
| A10 | `except: pass` | 구체적 예외 처리 |

#### 3.3 Java

| OWASP | 취약 패턴 | 안전한 패턴 |
|-------|-----------|-------------|
| A01 | `@GetMapping` 권한 검증 없음 | `@PreAuthorize` 추가 |
| A02 | `server.ssl.enabled=false` | TLS 활성화 |
| A04 | `MessageDigest.getInstance("MD5")` | `SHA-256` 또는 `bcrypt` |
| A04 | `new DESKeySpec()` | `AES/GCM/NoPadding` |
| A05 | `stmt.execute("..." + input)` | `PreparedStatement` |
| A05 | `Runtime.exec(cmd)` | `ProcessBuilder` + 화이트리스트 |
| A07 | 세션 고정 공격 취약 | `session.invalidate()` 후 재생성 |
| A08 | `ObjectInputStream.readObject()` | 화이트리스트 역직렬화 |
| A09 | `System.out.println()` | SLF4J/Logback |
| A10 | `catch(Exception e) { e.printStackTrace() }` | 일반 에러 응답 |

#### 3.4 Go

| OWASP | 취약 패턴 | 안전한 패턴 |
|-------|-----------|-------------|
| A01 | URL 파라미터 직접 쿼리 | 권한 검증 미들웨어 |
| A02 | `http.ListenAndServe` (no TLS) | `http.ListenAndServeTLS` |
| A04 | `md5.New()` | `sha256.New()` |
| A04 | `crypto/des` | `crypto/aes` |
| A05 | 문자열 연결 SQL | `db.Query("...?", param)` |
| A05 | `exec.Command("sh", "-c", userCmd)` | 명령어 화이트리스트 |
| A06 | 뮤텍스 없는 goroutine | `sync.Mutex` 사용 |
| A08 | `go get pkg@master` | 버전 고정 + checksum |
| A09 | `if err != nil {}` (무시) | 구조화 로깅 |
| A10 | 무한 루프 가능성 | 최대 반복/깊이 제한 |

---

### Phase 4: SKILL.md 작성

#### 4.1 구조
1. YAML frontmatter (name, description)
2. 스킬 목적 및 사용 시점
3. 리뷰 프로세스 가이드
4. OWASP Top 10 2025 체크리스트
5. 언어별 취약점 패턴 요약
6. 출력 형식 템플릿
7. 심각도 분류 기준

#### 4.2 핵심 기능
- 코드 리뷰 시 자동 보안 관점 적용
- 각 취약점에 대한 구체적 탐지 패턴 제공
- 발견된 취약점에 대한 수정 가이드 제공
- 심각도 기반 우선순위 제시

---

### Phase 5: 테스트 및 검증

#### 5.1 테스트 케이스
각 OWASP 카테고리별 취약한 코드 샘플로 테스트:
- JavaScript: Express.js 애플리케이션
- Python: Flask/Django 애플리케이션
- Java: Spring Boot 애플리케이션
- Go: net/http 애플리케이션

#### 5.2 검증 항목
- [ ] 각 취약점 카테고리 탐지 여부
- [ ] 오탐(False Positive) 비율
- [ ] 수정 제안의 정확성
- [ ] 출력 형식 일관성

---

### Phase 6: 문서화 및 배포

#### 6.1 README 업데이트
- 새 스킬 목록에 추가
- 사용 예시 추가

#### 6.2 커밋 및 PR
```
feat(skills): add security-review skill based on OWASP Top 10 2025

- Add comprehensive security review skill
- Support JS/TS, Python, Java, Go
- Include detection patterns for all OWASP categories
- Provide fix recommendations with code examples

Closes #1
```

---

## 일정 (작업 순서)

1. **Phase 1**: 기반 구조 설계
2. **Phase 2**: OWASP 상세 분석 (이 문서에서 완료)
3. **Phase 3**: 언어별 패턴 정의 (이 문서에서 완료)
4. **Phase 4**: SKILL.md 작성
5. **Phase 5**: 테스트 및 검증
6. **Phase 6**: 문서화 및 배포

---

## 산출물

| 파일 | 설명 |
|------|------|
| `skills/security-review/SKILL.md` | 메인 스킬 정의 |
| `README.md` (업데이트) | 스킬 목록에 추가 |
| 이슈 #1 종료 | PR 머지 후 자동 종료 |
