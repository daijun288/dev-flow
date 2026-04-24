# Agent Prompt Templates

---

## Check Agent Templates

### First Invocation

```json
{
  "subagent_type": "general-purpose",
  "description": "Comprehensive code check",
  "prompt": "Comprehensive code check, MUST NOT modify.\n\nRequirement summary:\n[Requirement]\n\nCheck scope (MUST check all, MUST NOT skip):\nModified files:\n[File list]\n\nAffected files:\n[File list]\n\n[MANDATORY Check Items]:\n1. Requirement compliance: Functionality completeness, parameter/field completeness, business logic correctness\n2. Code syntax: Syntax errors, type errors, import errors, undefined variables\n3. Security check: SQL injection, XSS, command injection, hardcoded sensitive info\n4. Code standards: Naming conventions, code style, consistency with existing code\n5. Affected impact: Whether modifications affect calls in affected files, interface compatibility, parameter matching\n\nOutput format:\nCheck result:\n- Requirement compliance: ✓/✗ [Explanation]\n- Code syntax: ✓/✗ [Explanation]\n- Security check: ✓/✗ [Explanation]\n- Code standards: ✓/✗ [Explanation]\n- Affected impact: ✓/✗ [Explanation]\n\nIf issues exist, list all issues:\n- File: [File path]\n- Type: [Modified file/Affected file]\n- Location: [Line number]\n- Issue: [Issue description]\n\nMUST NOT: Do not modify code, ONLY perform check analysis.\n\nRemember: SendMessage will notify modified files later, keep memory."
}
```

### SendMessage Reuse (After Fix)

```json
{
  "to": "[Agent_ID]",
  "prompt": "Fix content:\n[Fix description]\n\n[IMPORTANT]Please re-read the following files:\nModified files:\n[File list]\n\nPlease re-check."
}
```

---

## Affected Files Search Method

```bash
# Java
grep -r "UserService" --include="*.java" | grep -v "UserService.java"

# JavaScript/Vue
grep -r "api" --include="*.vue" --include="*.js" | grep -v "api.js"

# Python
grep -r "user_service" --include="*.py" | grep -v "user_service.py"
```