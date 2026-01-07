---
name: security-review
description: Perform security code reviews based on OWASP Top 10 2025. Use this skill when users request security review, vulnerability assessment, or ask to check their code for security issues. Supports JavaScript/TypeScript, Python, Java, and Go.
---

# Security Review Skill (OWASP Top 10 2025)

You are an expert application security engineer. Perform comprehensive security reviews based on OWASP Top 10 2025.

## Review Process

1. **Identify the language and framework** being used
2. **Scan for vulnerability patterns** in each OWASP category
3. **Assess severity** based on exploitability and impact
4. **Provide specific fixes** with secure code examples

## OWASP Top 10 2025 Checklist

### A01: Broken Access Control
- [ ] Authorization checks on all endpoints
- [ ] No direct object references without ownership validation
- [ ] SSRF prevention (URL allowlists)
- [ ] No privilege escalation paths

### A02: Security Misconfiguration
- [ ] Security headers present (CSP, HSTS, X-Frame-Options)
- [ ] Debug mode disabled in production
- [ ] No default credentials
- [ ] Error messages don't expose internals

### A03: Software Supply Chain Failures
- [ ] Dependencies pinned to specific versions
- [ ] No vulnerable dependencies (check CVEs)
- [ ] Lockfiles present and committed
- [ ] No dynamic package installation from user input

### A04: Cryptographic Failures
- [ ] Strong algorithms (SHA-256+, AES-GCM, bcrypt/argon2)
- [ ] No hardcoded secrets or keys
- [ ] Proper key management
- [ ] TLS for data in transit

### A05: Injection
- [ ] Parameterized queries for SQL
- [ ] No eval() or equivalent with user input
- [ ] Output encoding for XSS prevention
- [ ] Command injection prevention

### A06: Insecure Design
- [ ] Threat modeling considered
- [ ] Race condition protection
- [ ] Business logic validation
- [ ] Secure file upload handling

### A07: Authentication Failures
- [ ] Strong password policies
- [ ] Session management (regeneration, expiry)
- [ ] Rate limiting on auth endpoints
- [ ] MFA support where applicable

### A08: Software/Data Integrity Failures
- [ ] Safe deserialization practices
- [ ] Signed artifacts and updates
- [ ] CI/CD pipeline security
- [ ] No unsafe object hydration

### A09: Security Logging & Alerting Failures
- [ ] Authentication events logged
- [ ] Failed access attempts logged
- [ ] Structured logging format
- [ ] No sensitive data in logs

### A10: Mishandling Exceptional Conditions
- [ ] Generic error messages to users
- [ ] No stack traces in responses
- [ ] Resource limits (loops, recursion)
- [ ] Graceful degradation

---

## Vulnerability Patterns by Language

### JavaScript/TypeScript

| Category | Vulnerable Pattern | Secure Pattern |
|----------|-------------------|----------------|
| A01 | `req.params.id` used directly in DB | Validate `req.user.id` ownership |
| A02 | `cors({origin: '*'})` | `cors({origin: allowedOrigins})` |
| A04 | `crypto.createHash('MD5')` | `crypto.createHash('sha256')` |
| A05 | `db.query("SELECT * FROM x WHERE id=" + id)` | `db.query("SELECT * FROM x WHERE id=?", [id])` |
| A05 | `eval(userInput)` | `JSON.parse()` with validation |
| A05 | `element.innerHTML = userInput` | `element.textContent = userInput` |
| A07 | Hardcoded passwords | Environment variables / secrets manager |
| A08 | `JSON.parse()` → prototype pollution | Object.freeze or validation |
| A09 | `console.log()` only | winston/pino structured logging |
| A10 | `catch(e) { res.send(e.stack) }` | `catch(e) { res.status(500).send('Error') }` |

**Critical Patterns to Detect:**
```javascript
// A05: SQL Injection
db.query("SELECT * FROM users WHERE id = " + req.params.id);

// A05: Command Injection
exec("convert " + req.body.filename);

// A05: XSS
res.send("<div>" + req.query.name + "</div>");

// A04: Weak Crypto
crypto.createHash('md5').update(password).digest('hex');

// A01: SSRF
fetch(req.query.url);
```

### Python

| Category | Vulnerable Pattern | Secure Pattern |
|----------|-------------------|----------------|
| A01 | `Model.objects.get(id=request.GET['id'])` | Add ownership check |
| A02 | `app.run(debug=True)` | `app.run(debug=False)` |
| A04 | `hashlib.md5(password.encode())` | `bcrypt.hashpw(password, salt)` |
| A05 | `cursor.execute("..." + input)` | `cursor.execute("...%s", (input,))` |
| A05 | `os.system(user_cmd)` | `subprocess.run([...], shell=False)` |
| A05 | `render_template_string(user_input)` | `render_template()` with static templates |
| A07 | `if password == stored_password` | `bcrypt.checkpw()` |
| A08 | `pickle.loads(user_data)` | `json.loads()` with schema validation |
| A08 | `yaml.load(data)` | `yaml.safe_load(data)` |
| A10 | `except: pass` | Specific exception handling with logging |

**Critical Patterns to Detect:**
```python
# A05: SQL Injection
cursor.execute("SELECT * FROM users WHERE id = " + request.args['id'])

# A05: Command Injection
os.system("convert " + filename)

# A08: Unsafe Deserialization
data = pickle.loads(request.data)

# A05: SSTI
return render_template_string(request.args.get('template'))

# A04: Weak Hashing
hashlib.md5(password.encode()).hexdigest()
```

### Java

| Category | Vulnerable Pattern | Secure Pattern |
|----------|-------------------|----------------|
| A01 | `@GetMapping` without `@PreAuthorize` | Add authorization annotation |
| A02 | `server.ssl.enabled=false` | Enable TLS |
| A04 | `MessageDigest.getInstance("MD5")` | `BCrypt.hashpw()` |
| A04 | `new DESKeySpec(key)` | `AES/GCM/NoPadding` |
| A05 | `stmt.executeQuery("..." + input)` | `PreparedStatement` |
| A05 | `Runtime.getRuntime().exec(cmd)` | `ProcessBuilder` with allowlist |
| A07 | Session fixation vulnerable | `session.invalidate()` before new session |
| A08 | `ObjectInputStream.readObject()` | Allowlist deserialization filter |
| A09 | `System.out.println()` | SLF4J/Logback |
| A10 | `e.printStackTrace()` in response | Generic error message |

**Critical Patterns to Detect:**
```java
// A05: SQL Injection
Statement stmt = conn.createStatement();
stmt.executeQuery("SELECT * FROM users WHERE id = " + userId);

// A05: Command Injection
Runtime.getRuntime().exec("cmd /c " + userInput);

// A08: Unsafe Deserialization
ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
Object obj = ois.readObject();

// A04: Weak Crypto
MessageDigest md = MessageDigest.getInstance("MD5");

// A01: Missing Authorization
@GetMapping("/admin/users/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id);  // No auth check
}
```

### Go

| Category | Vulnerable Pattern | Secure Pattern |
|----------|-------------------|----------------|
| A01 | URL param direct to query | Authorization middleware |
| A02 | `http.ListenAndServe()` | `http.ListenAndServeTLS()` |
| A04 | `md5.New()` | `sha256.New()` |
| A04 | `crypto/des` | `crypto/aes` with GCM |
| A05 | String concatenation in SQL | `db.Query("...?", param)` |
| A05 | `exec.Command("sh", "-c", userCmd)` | Command allowlist |
| A06 | Goroutine without mutex | `sync.Mutex` for shared state |
| A08 | `go get pkg@master` | Pin versions in go.mod |
| A09 | `if err != nil { }` (silent) | Structured logging with zap/zerolog |
| A10 | Unbounded loops | Max iteration limits |

**Critical Patterns to Detect:**
```go
// A05: SQL Injection
db.Query("SELECT * FROM users WHERE id = " + r.URL.Query().Get("id"))

// A05: Command Injection
exec.Command("sh", "-c", userInput).Run()

// A04: Weak Hash
h := md5.New()
h.Write([]byte(password))

// A01: SSRF
resp, _ := http.Get(r.URL.Query().Get("url"))

// A06: Race Condition
go func() {
    balance[userID] += amount  // No mutex
}()
```

---

## Severity Classification

| Severity | Criteria | Examples |
|----------|----------|----------|
| **Critical** | Remote code execution, auth bypass, data breach | SQL injection, command injection, broken auth |
| **High** | Significant data exposure, privilege escalation | XSS, SSRF, insecure deserialization |
| **Medium** | Limited impact, requires specific conditions | Security misconfig, weak crypto in non-critical paths |
| **Low** | Informational, defense in depth | Missing headers, verbose errors |

---

## Output Format

Structure your security review as follows:

```markdown
## Security Review Report

### Summary
| Severity | Count |
|----------|-------|
| Critical | X |
| High | X |
| Medium | X |
| Low | X |

### Findings

#### [CRITICAL] A05: SQL Injection
**Location**: `src/api/users.js:42`
**Vulnerable Code**:
```javascript
db.query("SELECT * FROM users WHERE id = " + req.params.id);
```
**Risk**: Attacker can execute arbitrary SQL queries, leading to data breach or deletion.
**Recommendation**:
```javascript
db.query("SELECT * FROM users WHERE id = ?", [req.params.id]);
```

#### [HIGH] A01: Broken Access Control
**Location**: `src/api/orders.js:28`
...

### Recommendations
1. Prioritize Critical and High severity issues
2. Implement input validation layer
3. Add security headers middleware
4. Enable structured logging for security events
```

---

## Guidelines

- **Be thorough**: Check every user input flow
- **Be specific**: Reference exact file paths and line numbers
- **Be actionable**: Provide working secure code examples
- **Prioritize**: Focus on Critical/High issues first
- **Consider context**: Framework-specific security features may already mitigate some risks
- **Check dependencies**: Note if vulnerable packages are detected
