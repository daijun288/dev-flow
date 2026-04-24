---
name: dev-flow
description: One-stop code change workflow.
argument-hint: [requirement description]
disable-model-invocation: true
allowed-tools: Read Grep Glob Edit Write Bash(mvn *) Bash(npm *) Bash(git *) AskUserQuestion Agent
---

# dev-flow - One-stop Code Change Workflow

---

## Core Design: Check Agent to Avoid Self-Bias

**Use independent Check Agent for code review phase. Main Agent handles coding and coordination.**

| Agent | Responsibility | Lifecycle |
| :--- | :--- | :--- |
| Check Agent | Code review, checking modified files + affected files | Start BEFORE coding → Release when outputting results |

---

## Mandatory Requirements

| Requirement | Description |
| :--- | :--- |
| **MUST NOT pause midway** | After user confirmation, MUST execute all steps continuously |
| **MUST NOT skip testing** | Compilation/testing phase MUST be executed |
| **Check Agent MUST start BEFORE coding** | Start before coding, NOT after |
| **ROLLBACK MUST reuse Agent** | After rollback, MUST use SendMessage to reuse Check Agent, MUST NOT create new one |

---

## Phase 1: Requirement Confirmation

### Flow

Main Agent reads code → Analyzes gaps using checklist → AskUserQuestion → User answers → Continue or end

Main Agent MUST:
1. Read related code to understand current implementation
2. Check against gaps checklist in requirement.md
3. Call AskUserQuestion with N question objects
4. After receiving answers, if gaps remain, continue asking
5. If no gaps, output requirement summary and ask user to confirm

[MANDATORY] AskUserQuestion format: see [reference/requirement.md](reference/requirement.md)

---

## Phase 2: Coding

### Flow

```
Start Check Agent (BEFORE coding!)
    ↓
Main Agent performs coding
```

[MANDATORY] Check Agent MUST start BEFORE coding begins.

---

## Phase 3: Check

### Flow

```
SendMessage(Check Agent) → Check code issues
    ↓
┌───────────────────────────────────────────────┐
│ Check Result?                                 │
│   ├─ No issues → Enter compilation testing    │
│   └─ Has issues → ⏪ ROLLBACK to coding       │
└───────────────────────────────────────────────┘
```

[MANDATORY] SendMessage MUST include the list of modified files.

See: [reference/validation.md](reference/validation.md)

---

## Phase 4: Compilation Testing

### Flow

```
Main Agent executes compilation testing
    ↓
┌───────────────────────────────────────────────┐
│ Test Result?                                  │
│   ├─ Pass → Enter output results              │
│   └─ Fail → ⏪ ROLLBACK to coding             │
└───────────────────────────────────────────────┘
```

[MANDATORY] MUST NOT skip testing.

See: [reference/validation.md](reference/validation.md) for test commands

---

## Phase 5: Output Results

```
Release Check Agent
    ↓
Output task complete
```

---

## Rollback Mechanism

```
Check/Test fail → ⏪ ROLLBACK to coding → SendMessage(Check Agent_ID) → Re-execute check→test
```

See: [reference/validation.md](reference/validation.md) for rollback declaration format

---

## User Commands

| Command | Effect |
| :--- | :--- |
| /abort | Abort task |
| 够了/确认/开始/OK | Stop questioning, start coding |

---

## Agent Invocation Rules

| Rule | Description |
| :--- | :--- |
| Before calling Agent | MUST NOT output text, call directly |
| After calling Agent | Wait for result directly, MUST NOT output "waiting" text |
| If AskUserQuestion returns duplicate answers | Use only the first answer, ignore duplicates |
| SendMessage | MUST include list of modified files |
| Agent release | Release after entire workflow completes |

See: [reference/agent-templates.md](reference/agent-templates.md)

---

## Resource Navigation

- **Requirement Confirmation Rules**: [reference/requirement.md](reference/requirement.md)
- **Check→Test Rules**: [reference/validation.md](reference/validation.md)
- **Agent Templates**: [reference/agent-templates.md](reference/agent-templates.md)

---

$ARGUMENTS
