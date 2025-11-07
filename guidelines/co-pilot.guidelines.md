---
# Copilot Guidelines (Repository)

Treat Copilot as an assistant — a junior developer that speeds up routine work but requires human review.

This document collects pragmatic rules, prompting tips, and safety checks we follow when using Copilot or other code-generation tools in this repo.

---
## 1. High-level rules

- Use Copilot for low-risk, repetitive tasks: http client calls, publishing/consuming from kafka, Controller scaffolds,DTOs, POJOs, mappers, test scaffolding, documentation snippets.
- Don't use Copilot for security-sensitive logic (auth, crypto, secret handling), complex business rules, or unfamiliar frameworks without review.
- Always run compilation, linting, and tests on any Copilot-generated code before merging.

---
## 2. When to use it

- Boilerplate (DTOs, request/response models, simple mappers)
- Unit and integration test scaffolding (write the test name and expected input/output clearly)
- Small refactors and helper extraction
- API exploration (ask Copilot Chat for examples), but validate the suggestions
- http client calls, kafka publisher/consumer, controller/repository scaffold

---
## 3. When not to use it

- Authentication, authorization, encryption or any secrets handling
- Complex business workflows or domain rules that require context-specific judgement
- Large “Vibe coding” patches — generate smaller focused changes instead

---
## 4. Prompting best practices

- Always provide intent before the code (a short comment or method signature).
- Give a small example when possible (input -> expected output).
- Keep context minimal and current — remove outdated comments that may confuse completions.

Example:

```java
// Validate user credentials and return JWT if valid
public String authenticateUser(String username, String password) {
    // ...
}
```

---
## 5. Developer productivity patterns

| Goal                 | Tip                                                          |
| -------------------- | ------------------------------------------------------------ |
| Generate boilerplate  | Use Copilot for DTOs, mappers, request/response, flyway migrations, scaffolds 
| Explore APIs          | Ask Copilot Chat for examples of SDK or library usage        |
| Improve readability   | Use Copilot to refactor long methods into helpers            |
| Speed up reviews      | Use Copilot Chat: “Explain this diff” or “Summarize changes” |
| Write docs            | Generate function docstrings from code blocks                |

---
## 6. Common gotchas & quick fixes

| Issue                  | Fix                                                 |
| ---------------------- | --------------------------------------------------- |
| Wrong API calls        | Add import + short comment before code              |
| Overly verbose code    | Ask Copilot Chat: “Make it more concise”            |
| Repetitive suggestions | Clear context (remove unrelated comments)           |
| Wrong test assertions  | Write test name and example input/output explicitly |
| Security risks         | Run code scans and a security review                |

---
## 7. Code review & verification checklist

Before merging Copilot output, ensure:

- Code compiles and tests pass
- Lint and static analysis checks are green
- Naming, logging and validation follow project conventions
- No secrets or hard-coded credentials
- A human reviewer has checked domain correctness

---
## 8. Learning & feedback loop

- Maintain a short feedback loop: capture good/bad examples, update prompts, and share guidance in this file.
- Periodically review Copilot suggestions against team conventions and update these guidelines accordingly.

---
## 9. Closing note

Copilot accelerates engineers when used with discipline. Treat its suggestions as drafts — not as final production code.

> "AI assistance is powerful, but human judgment remains irreplaceable."
