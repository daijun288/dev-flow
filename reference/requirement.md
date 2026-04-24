# Requirement Confirmation Phase

---

## Core Principle

Main Agent reads code, identifies gaps using checklist, and asks user directly via AskUserQuestion.

---

## Flow

```
Read code → Check gaps checklist → AskUserQuestion → Receive answers → Continue or summarize
```

---

## [MANDATORY] AskUserQuestion Format Per Round

When Main Agent identifies N gaps, MUST call AskUserQuestion ONCE with N question objects in the questions array.

Each question object MUST have:
- question: "Issue X/N: [gap description]"
- header: short topic label
- options: at least 2 options specific to that gap
- multiSelect: true or false, determined by the nature of the gap

Correct format:
```json
{
  "questions": [
    {
      "question": "Issue 1/N: [gap description]",
      "header": "[topic]",
      "options": [
        {"label": "Option A", "description": "description of option A"},
        {"label": "Option B", "description": "description of option B"}
      ],
      "multiSelect": false
    },
    {
      "question": "Issue 2/N: [gap description]",
      "header": "[topic]",
      "options": [
        {"label": "Option A", "description": "description of option A"},
        {"label": "Option B", "description": "description of option B"}
      ],
      "multiSelect": true
    }
  ]
}
```

MUST NOT use a single question that batches all gaps:
```json
{
  "questions": [
    {
      "question": "How to handle all N gaps?",
      "options": [
        {"label": "Fix all", "description": "..."},
        {"label": "Ignore all", "description": "..."}
      ]
    }
  ]
}
```

---

## [MANDATORY] After Receiving Answers

Step 1: Parse user answers
Step 2: MUST call AskUserQuestion ONCE to ask "Any supplement or changes?" with options:
  - "No supplement, proceed"
  - "I have supplements"
Step 3: If user has supplements → Merge and continue loop
Step 4: If user says proceed → Output requirement summary
Step 5: After summary, MUST ask user to confirm before coding
Step 6: Only start coding after user says "确认"/"开始"/"OK"

MUST NOT skip directly from answers to summary without asking for supplement.
MUST NOT start coding without user confirmation of summary.

---

## Loop End Conditions

| Condition | Action |
| :--- | :--- |
| User halt (够了/确认/开始/OK) | End loop → Coding |
| No gaps + User confirms | End loop → Coding |
| Has gaps | AskUserQuestion → Continue loop |
| No gaps + User has supplement | Merge summary → Continue loop |

---

## User Halt Keywords

| Keyword | Effect |
| :--- | :--- |
| 够了 | Stop questioning, end loop |
| 确认 | Stop questioning, end loop |
| 开始 | Stop questioning, end loop |
| OK | Stop questioning, end loop |
| /abort | Abort task |

---

## Main Agent MUST NOT Behaviors

| MUST NOT | Correct |
| :--- | :--- |
| Skip reading code before asking | Read first, then ask |
| Output "waiting" text | Call directly |
| Use single batched question | N questions in one AskUserQuestion |
| End without user confirmation | Ask user to confirm summary |
| Call AskUserQuestion multiple times per round | One AskUserQuestion per round with N questions |

---

## Requirement Summary Format

**Use when summarizing requirements:**

```
需求摘要：
- 目标: [一句话描述]
- 改动范围: [预估涉及的文件/模块]
- 关键点:
  - [第1轮确认的内容]
  - [第2轮确认的内容]（如有）
```

---

## Gaps Checklist

| Type | Gaps |
| :--- | :--- |
| Feature addition | Permissions, validation, concurrency, transactions, exception handling |
| API endpoint | Parameter validation, error handling, response format, idempotency |
| UI interface | Loading/empty/error states, responsive, interaction feedback |
| Data collection | Failure handling, format compatibility, performance, storage |
| Backend service | Logging, monitoring, config, health check, degradation |