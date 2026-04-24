# Check → Test → Acceptance Phase - Check Agent Rules

---

## Core Principle

**Check Agent participates throughout entire phase. Any phase failure triggers ROLLBACK to coding and reuse via SendMessage.**

---

## Flow Diagram

```
Coding → SendMessage(Check Agent) → Check result
    ↓
┌─────────────────────────────────────────────────────┐
│ Check → Issues?                                     │
│   ├─ No issues → Enter compilation testing          │
│   └─ Has issues → Output rollback declaration → ROLLBACK to coding│
│                ↓                                    │
│          SendMessage(Agent_ID) → Re-check           │
└─────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────┐
│ Compilation testing → Result?                       │
│   ├─ Pass → Enter output results                    │
│   └─ Fail → Output rollback declaration → ROLLBACK to coding│
│                ↓                                    │
│          SendMessage(Agent_ID) → Check→Test         │
└─────────────────────────────────────────────────────┘
```

---

## Check Agent Invocation

### Input Scope

| File Type | Description |
| :--- | :--- |
| Modified files | Files edited/written by Main Agent this time |
| Affected files | Files that depend on modified files |

**Find affected files**:
```bash
# Java
grep -r "UserService" --include="*.java" | grep -v "UserService.java"

# JavaScript/Vue
grep -r "api" --include="*.vue" --include="*.js" | grep -v "api.js"

# Python
grep -r "user_service" --include="*.py" | grep -v "user_service.py"
```

### Check Items (MANDATORY)

1. Requirement compliance: Complete functionality, complete parameters, correct logic
2. Code syntax: Syntax/type/import/undefined errors
3. Security check: SQL injection, XSS, command injection, hardcoded sensitive info
4. Code standards: Naming, style, consistency
5. Affected impact: Call compatibility, interface matching

---

## SendMessage Reuse

**MUST include modified files list:**

```json
{
  "to": "[Agent_ID]",
  "prompt": "Fix content:\n[Fix description]\n\n[IMPORTANT]Please re-read the following files:\nModified files:\n[File list]\n\nPlease re-check."
}
```

---

## Rollback Declaration Format

### Check Failed

```
⏪ ROLLBACK: Check phase found issues, return to coding phase for fix

Found [N] issues:
1. [File path] Line [L] - [Issue description]
2. [File path] Line [L] - [Issue description]

After fix execute: SendMessage(Check Agent_ID) → Check→Test
```

### Test Failed

```
⏪ ROLLBACK: Test phase failed, return to coding phase for fix

Test failure reason:
- [Test name]: [Error message]

After fix execute: SendMessage(Check Agent_ID) → Check→Test
```

---

## Test Commands (MANDATORY Execution)

| Project Type | Command |
| :--- | :--- |
| Maven Java | `mvn test` |
| npm Node.js | `npm test` |
| pytest Python | `pytest` |
| Go | `go test ./...` |

**MUST NOT skip testing.**

---

## Check Agent Lifecycle

| Timing | Action |
| :--- | :--- |
| Before coding | Start Check Agent |
| When outputting results | Release Check Agent |

**Any phase failure, DO NOT release Agent, use SendMessage to reuse.**

---

## Main Agent MUST NOT Behaviors

| MUST NOT | Correct |
| :--- | :--- |
| Output text before calling Agent | Call directly, no text |
| Output "waiting for result" after calling Agent | Wait directly, process result immediately |
| Judge whether code has issues yourself | Let Check Agent judge, ONLY handle fixing |
| Release Agent after failure | Keep Agent, use SendMessage to reuse |

---

## Agent Templates

See: [agent-templates.md](agent-templates.md) Check Agent section