# chaos-qa-hunter

**A Claude Code skill for adversarial, end-to-end bug hunting.**

This skill turns a Claude agent into a pure **bug finder** — it reads every line of your source code (white-box), attacks the system from a real user's perspective, and documents every error in a structured `BUGS.md` file. It **never modifies code** and **never fixes anything**. Its only job is to break things and write it down.

---

## What it does

- **White-box + end-to-end**: reads all source code first, then uses that knowledge to attack the system like a real user
- **7 attack vectors**: boundary values, state machine abuse, concurrency, missing fields, injection attacks, large data, normal flow edge cases
- **Zero fixing**: the Iron Law — no code modification, ever
- **95% coverage loop**: keeps hunting until 5 conditions are met; documents why it stopped
- **Structured BUGS.md**: every bug includes severity, exact repro steps, precise input values, full error output, and code location — so a second agent can reproduce and fix without guessing

---

## Install

Copy `SKILL.md` into your Claude Code personal skills directory:

**macOS / Linux:**
```bash
mkdir -p ~/.claude/skills/chaos-qa-hunter
curl -o ~/.claude/skills/chaos-qa-hunter/SKILL.md \
  https://raw.githubusercontent.com/liboze2026/chaos-qa-hunter/main/SKILL.md
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills\chaos-qa-hunter"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/liboze2026/chaos-qa-hunter/main/SKILL.md" `
  -OutFile "$env:USERPROFILE\.claude\skills\chaos-qa-hunter\SKILL.md"
```

---

## Usage

In any Claude Code session, type:

```
/chaos-qa-hunter
```

Claude will:
1. Read all source files and build a coverage map
2. Systematically attack the system across 7 vectors
3. Record each bug to `BUGS.md` as it's found
4. Loop until ≥95% coverage is confirmed
5. Output a final `BUGS.md` ready for a fix agent

---

## The Iron Law

The agent using this skill **cannot**:
- Modify any source file
- Fix any error it finds
- Suggest fixes in the bug report
- Stop before reaching 95% coverage

---

## 95% Stop Criteria

The skill only stops when **all five** conditions are met:

| Condition | Threshold |
|-----------|-----------|
| Function coverage | ≥ 95% |
| Branch coverage (if/else) | ≥ 90% |
| All P0 attack vectors applied to all P0 entry points | 100% |
| Two consecutive rounds with zero new High/Critical bugs | — |
| All error-handling paths triggered at least once | 100% |

---

## BUGS.md Output Format

Each bug entry includes:

```markdown
## BUG-001: [One-line description]
- Severity: Critical / High / Medium / Low
- Type: Crash / Logic / Security / UX / Data / Performance
- Repro steps: exact numbered steps
- Precise input: JSON/command with exact values
- Expected: what should happen
- Actual: what happened (full error + stack trace)
- Code location: file:line
- Triggered path: call chain from entry to failure
- Attack vector: which of the 7 vectors triggered this
```

---

## Attack Vectors

| Vector | Examples |
|--------|----------|
| Boundary values | `0`, `-1`, `MAX_INT+1`, `""`, `" "`, 10,000-char string |
| State machine | skip steps, repeat steps, reverse order, concurrent ops |
| Missing fields | omit required fields, send `null`, send wrong type |
| Injection | SQL, XSS, path traversal, template injection, null bytes |
| Concurrency | simultaneous debit of same balance, double-submit |
| Large data | oversized files, deeply nested JSON, 1000-item lists |
| Normal flow | happy path edge cases, flash states, empty states |

---

## License

MIT
