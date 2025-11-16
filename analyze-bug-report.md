---
description: Deeply analyze a bug report - validity, priority, and fix recommendation
argument-hint: "<bug report text>"
model: claude-sonnet-4-20250514
allowed-tools:
  - Read
  - Bash(git *)
  - Bash(gh *)
  - Bash(grep *)
  - mcp__*
---

# Deep Bug Analysis

## Report:
$ARGUMENTS

---

Think carefully through this bug report using extended reasoning. Use MCP tools when needed. Use sequential thinking if needed.

## Analyze:

**1. VALIDITY**
- Is this actually a bug or expected behavior?
- Is there enough information to reproduce it?
- Search the codebase for related functionality if needed

**2. PRIORITY** (Think through deeply)
- Severity: How bad is it? (Critical/High/Medium/Low)
- Frequency: How often does this happen?
- Impact: How many users affected?
- Business risk: Revenue, security, compliance issues?

**3. SOLUTION**
- What's the root cause?
- What's the best fix?
- What are the risks?
- Which files need changes?
- How do we test it?

## Deliver:

**VERDICT**: Valid Bug | Not a Bug | Needs More Info

**PRIORITY**: P0/P1/P2/P3 with reasoning

**RECOMMENDED FIX**: 
- Root cause explanation
- Solution approach
- Files to change
- Testing strategy
- Estimated complexity

Use deep thinking to reason through technical implications and edge cases.
