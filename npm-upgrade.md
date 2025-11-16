---
description: Intelligently upgrade npm packages with changelog analysis and risk assessment
argument-hint: "[--safe-only | --dry-run | --package <name>]"
model: claude-sonnet-4-20250514
allowed-tools:
  - Read
  - Write
  - Bash(*)
  - web_search
  - web_fetch
  - mcp__*
---

# Intelligent NPM Package Upgrader

**Arguments**:
- No args: Analyze all packages
- `--safe-only`: Only analyze and upgrade safe packages
- `--dry-run`: Analysis only, no upgrades
- `--package <name>`: Analyze specific package only

**Target**: $ARGUMENTS

---

## Pre-flight
```bash
# Save current state
!git add package.json package-lock.json 2>/dev/null || true
!git diff --staged --quiet || git commit -m "Checkpoint: before npm upgrade analysis"

# Read current state
!cat package.json | grep -A 999 '"dependencies"'
!npm outdated --json 2>/dev/null || npm outdated
```

## Discovery Phase

Identify all outdated packages and their version jumps:
```bash
!npm outdated --long --json
```

Parse results to extract:
- Package name
- Current version
- Wanted version (semver-compatible)
- Latest version
- Package type (prod vs dev dependencies)
- Location in dependency tree

## Analysis Phase

For each package (or just the one specified in $ARGUMENTS):

### 1. Fetch Package Info
```bash
!npm view <package>@latest repository homepage description
!npm view <package>@<old-version> --json > /tmp/old.json
!npm view <package>@<new-version> --json > /tmp/new.json
```

### 2. Calculate Version Jump
```
Semver analysis:
- Patch: X.Y.Z → X.Y.Z+n (Risk: 1/10)
- Minor: X.Y.* → X.Y+n.* (Risk: 3/10)
- Major: X.*.* → X+n.*.* (Risk: 7/10)
- Multi-major: X.*.* → X+2+.*.* (Risk: 9/10)
```

### 3. Search for Changelog

You can use firecrawl mcp instead of web_*, if available

Try in order:
1. GitHub releases page
2. Raw CHANGELOG.md from GitHub
3. NPM package README
4. Package homepage
5. Git log comparison

Search query format:
```
<package-name> changelog <old-version> <new-version>
<package-name> breaking changes <new-version>
<package-name> migration guide <old-version> to <new-version>
```

### 4. Parse Changelog

Extract using patterns:
- `## [version]` or `# version`
- Look for: BREAKING, breaking:, ⚠️, 🚨, ⛔
- Look for: "feat:", "fix:", "chore:"
- Count commits between versions
- Find "Migration" or "Upgrade" sections

### 5. Security Check
```bash
!npm audit --json | jq '.vulnerabilities[] | select(.via[].name == "<package>")'
```

### 6. Dependency Impact
```bash
!npm ls <package> --json
!npm view <package>@<new-version> peerDependencies
```

Check for:
- Peer dependency changes
- New transitive dependencies
- Potential conflicts

### 7. Calculate Risk Score
```python
risk = 0

# Version jump
if major_version_change:
    risk += 5
    if major_jump > 1:
        risk += 2
elif minor_version_change:
    risk += 2
else:  # patch
    risk += 1

# Breaking changes
if "BREAKING" in changelog:
    risk += 3
if "migration" in changelog:
    risk += 1

# Size and maturity
if commits_between > 100:
    risk += 1
if package_age < 1_year:
    risk += 1

# Mitigating factors
if fixes_security_issue:
    risk -= 2
if has_migration_guide:
    risk -= 1
if comprehensive_tests:
    risk -= 1
if is_dev_dependency:
    risk -= 1

risk = max(0, min(10, risk))
```

### 8. Generate Recommendation
```
Risk 0-3:   ✅ Safe - Upgrade immediately
Risk 4-6:   ⚠️ Caution - Test thoroughly
Risk 7-10:  🛑 High Risk - Plan carefully

Priority modifiers:
- Security fix: URGENT (always recommend upgrade)
- Dev dependency: Less critical
- Core framework: More careful
```

## Output Report

For each package:
```
╔══════════════════════════════════════════════════════╗
║ 📦 <package-name>                                     ║
╠══════════════════════════════════════════════════════╣
║ Current:  v<x.y.z>                                    ║
║ Latest:   v<a.b.c>                                    ║
║ Jump:     [Patch/Minor/Major/Multi-Major]            ║
║ Risk:     [●●●○○○○○○○] 3/10                          ║
╠══════════════════════════════════════════════════════╣
║ CHANGES (<n> commits)                                 ║
╠══════════════════════════════════════════════════════╣
║ ✨ <new-features>                                     ║
║ 🐛 <bug-fixes>                                        ║
║ ⚠️ <breaking-changes>                                 ║
║ 🔒 <security-fixes>                                   ║
╠══════════════════════════════════════════════════════╣
║ RECOMMENDATION: [✅/⚠️/🛑]                              ║
╠══════════════════════════════════════════════════════╣
║ <reasoning>                                           ║
║                                                       ║
║ Action: <what-to-do>                                  ║
║                                                       ║
║ 🔗 Changelog: <url>                                   ║
║ 📖 Migration: <url-if-exists>                         ║
╚══════════════════════════════════════════════════════╝
```

## Final Summary
```
╔══════════════════════════════════════════════════════╗
║               UPGRADE PLAN                            ║
╠══════════════════════════════════════════════════════╣
║ 🟢 Safe (X packages)                                  ║
║ 🟡 Caution (Y packages)                               ║
║ 🔴 High-Risk (Z packages)                             ║
║ 🚨 Security Urgent (N packages)                       ║
╠══════════════════════════════════════════════════════╣
║ PHASE 1: Quick Wins (Do now)                          ║
╠══════════════════════════════════════════════════════╣
║ npm install \                                         ║
║   package1@x.y.z \                                    ║
║   package2@a.b.c                                      ║
╠══════════════════════════════════════════════════════╣
║ PHASE 2: Moderate Changes (This sprint)               ║
╠══════════════════════════════════════════════════════╣
║ npm install package3@latest                           ║
║ # Then: npm test && review migration notes            ║
╠══════════════════════════════════════════════════════╣
║ PHASE 3: Major Updates (Plan dedicated time)          ║
╠══════════════════════════════════════════════════════╣
║ <package4>: Create branch, extensive testing          ║
║ <package5>: May require code refactoring              ║
╚══════════════════════════════════════════════════════╝
```

## Auto-execution

If `--safe-only` was passed in $ARGUMENTS:
```bash
# Only upgrade packages with risk ≤ 3
!npm install <safe-package-1>@<version> <safe-package-2>@<version>

# Verify
!npm test
!npm run build

# Show diff
!git diff package.json package-lock.json
```

Otherwise, await user confirmation before executing.

