---
description: Analyze all GitHub PR comments as potential bug reports with deep analysis
argument-hint: "<PR URL or PR number>"
model: claude-sonnet-4-20250514
allowed-tools:
  - Read
  - Bash(gh *)
  - Bash(git *)
  - SlashCommand
  - mcp__github__get_pull_request
  - mcp__github__list_review_comments
  - mcp__github__list_issue_comments
---

# GitHub PR Comment Deep Analyzer

## Target PR
$ARGUMENTS

---

## Execution Plan

### 1️⃣ Fetch PR Information

First, I'll determine the best method to fetch PR data:

**Option A: GitHub CLI (preferred for simplicity)**
```bash
!gh pr view <NUMBER> --json title,body,comments,reviews,url
```

**Option B: MCP GitHub Server (if available)**
- Use MCP tools to fetch PR and comments
- Check with `/mcp` if github server is connected

**Option C: Git + API (fallback)**
```bash
!git remote get-url origin
# Parse to get owner/repo, then use gh api
```

### 2️⃣ Extract and Filter Comments

Parse all comments and identify potential bug reports by looking for:
- Keywords: "bug", "error", "broken", "fails", "doesn't work", "issue", "problem"
- Error messages or stack traces
- Unexpected behavior descriptions
- Reproducible steps
- "Should" vs "Actually" patterns

Categorize comments:
- 🐛 **Bug Reports**: Clear issue descriptions
- ⚠️ **Potential Issues**: Concerns or questions about behavior
- 💬 **Discussion**: General feedback (skip these)
- ✅ **Approvals**: LGTM, looks good (skip these)

### 3️⃣ Iterative Deep Analysis

For each bug report or potential issue, invoke:
```
/analyze-bug-report 

Comment from @<username> on <file>:<line>
PR: <PR title>
Context: <surrounding discussion if relevant>

<Full comment text>
```

**Present each analysis:**
```
═══════════════════════════════════════════════════════
📝 COMMENT #<N> - @<username> (<timestamp>)
File: <filename>:<line>
───────────────────────────────────────────────────────
<Original Comment>
───────────────────────────────────────────────────────
<Analysis from /analyze-bug-report>
═══════════════════════════════════════════════════════
```

### 4️⃣ Aggregate and Prioritize

After all analyses, create a consolidated report:

## 📊 PR ANALYSIS SUMMARY

**PR**: <title> (<URL>)
**Total Comments Analyzed**: <N> bug-related out of <M> total

### 🚨 Critical Findings (Requires Immediate Attention)
| Priority | Comment | Issue | Recommendation | Complexity |
|----------|---------|-------|----------------|------------|
| P0 | @user | [Brief desc] | [Fix approach] | [Simple/Medium/Complex] |

### ⚠️ Medium Priority Issues
| Priority | Comment | Issue | Recommendation |
|----------|---------|-------|----------------|
| P2 | @user | [Brief] | [Fix] |

### ℹ️ Low Priority / Non-Issues
- Comment from @user: [Brief reason why low priority or invalid]

### 📋 Recommended Actions

**Before Merging:**
- [ ] [Critical fix #1]
- [ ] [Critical fix #2]

**Schedule for Follow-up:**
- [ ] [Medium priority item #1]
- [ ] [Medium priority item #2]

**Consider but Not Blocking:**
- [ ] [Low priority improvements]

**PR Health Assessment**: 
- ✅ **Ready to merge** (after addressing critical items)
- ⚠️ **Needs work** (significant issues found)
- 🛑 **Not ready** (critical unresolved issues)

**Confidence Level**: [High/Medium/Low] based on information available in comments

---

## Prerequisites

This command works best with one of:
1. **GitHub CLI**: `gh auth login` (recommended)
2. **MCP GitHub Server**: Connected via `/mcp`
3. **Manual paste**: Paste PR comments directly if tools unavailable

## Example Usage
```bash
# With full URL
/analyze-pr-comments https://github.com/gumball-fm/platform/pull/1234

# With just PR number (uses current repo)
/analyze-pr-comments 1234

# If gh/MCP unavailable, I'll ask you to paste the comments
```

---

## Tips

- **Large PRs**: This can take 5-10 minutes for PRs with many comments
- **Rate Limits**: GitHub API has rate limits; command will pace appropriately
- **Context**: The more context in comments (file/line references), the better the analysis
- **Follow-up**: After analysis, you can ask me to dig deeper into specific findings
