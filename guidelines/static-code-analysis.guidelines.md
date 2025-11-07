# Spring Boot Static Code Analysis Setup

This setup enforces static code analysis and security checks in Gradle builds.

## 🚀 Tools Used
- **Checkstyle** – Style & convention checks
- **PMD** – Code smells + Method/Class length limits
- **SpotBugs + FindSecBugs** – Bug & security flaw detection
- **OWASP Dependency Check** – Detect vulnerable dependencies
- **JaCoCo** – Code coverage (fails if < 90%)

## 🔧 Failing Build Conditions
| Category | Rule |
|-----------|------|
| Method length | > 30 lines |
| Class length | > 150 lines |
| Code coverage | < 90% |
| Security CVE | CVSS ≥ 7 |
| Any static violation | Build fails |

## 🧩 Run Commands
```bash
./gradlew clean check
```

Reports generated in:
- `build/reports/checkstyle/main.html`
- `build/reports/pmd/main.html`
- `build/reports/spotbugs/main.html`
- `build/reports/dependency-check/`
- `build/reports/jacoco/test/html/index.html`
