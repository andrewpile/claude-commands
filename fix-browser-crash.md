---
description: Iteratively detect and fix browser rendering and console errors at localhost:8080
model: claude-sonnet-4-20250514
allowed-tools:
  - Read
  - Write
  - Bash(*)
  - mcp__control_chrome__*
---

# Browser Crash & Error Auto-Fixer

**Target**: http://localhost:8080
**Max Iterations**: 5
**Strategy**: Detect → Analyze → Fix → Verify → Repeat

---

## Execution Flow

### 🚀 Initialization

1. **Open Chrome tab to localhost:8080**
   - Use Control Chrome to navigate
   - Wait for initial page load

2. **Install comprehensive error monitoring**

Execute in browser:
```javascript
(function() {
  // Global error collector
  window.__diagnostics = {
    errors: [],
    warnings: [],
    networkErrors: [],
    renderIssues: [],
    startTime: Date.now()
  };

  // Console errors
  const origError = console.error;
  console.error = function(...args) {
    window.__diagnostics.errors.push({
      type: 'console.error',
      message: args.map(a => typeof a === 'object' ? JSON.stringify(a) : String(a)).join(' '),
      timestamp: Date.now(),
      stack: new Error().stack
    });
    origError.apply(console, args);
  };

  // Console warnings
  const origWarn = console.warn;
  console.warn = function(...args) {
    window.__diagnostics.warnings.push({
      type: 'console.warn',
      message: args.map(a => typeof a === 'object' ? JSON.stringify(a) : String(a)).join(' '),
      timestamp: Date.now()
    });
    origWarn.apply(console, args);
  };

  // Unhandled errors
  window.addEventListener('error', function(e) {
    window.__diagnostics.errors.push({
      type: 'unhandled',
      message: e.message,
      filename: e.filename,
      lineno: e.lineno,
      colno: e.colno,
      stack: e.error?.stack
    });
  }, true);

  // Promise rejections
  window.addEventListener('unhandledrejection', function(e) {
    window.__diagnostics.errors.push({
      type: 'promise',
      message: String(e.reason?.message || e.reason),
      stack: e.reason?.stack
    });
  });

  // Resource load failures
  window.addEventListener('error', function(e) {
    if (e.target !== window) {
      window.__diagnostics.networkErrors.push({
        type: 'resource',
        tag: e.target.tagName,
        src: e.target.src || e.target.href
      });
    }
  }, true);

  // Network failures (fetch/XHR)
  const origFetch = window.fetch;
  window.fetch = function(...args) {
    return origFetch.apply(this, args).catch(err => {
      window.__diagnostics.networkErrors.push({
        type: 'fetch',
        url: args[0],
        error: err.message
      });
      throw err;
    });
  };

  // Rendering checks
  setTimeout(() => {
    const issues = [];
    if (!document.body) issues.push('No body element');
    if (document.body?.children.length === 0) issues.push('Empty body');
    if (document.documentElement.scrollHeight < 50) issues.push('Suspiciously short page');
    
    const bodyStyle = window.getComputedStyle(document.body);
    if (bodyStyle.display === 'none') issues.push('Body is hidden');
    if (bodyStyle.opacity === '0') issues.push('Body is invisible');
    
    window.__diagnostics.renderIssues = issues;
  }, 1000);

  return 'Monitoring installed';
})();
```

### 🔄 Fix Iteration Loop

**Repeat up to 5 times or until clean:**

#### Step A: Collect Diagnostics

Execute in browser:
```javascript
(function() {
  const diag = window.__diagnostics || { errors: [], warnings: [], networkErrors: [], renderIssues: [] };
  
  return JSON.stringify({
    errors: diag.errors,
    warnings: diag.warnings,
    networkErrors: diag.networkErrors,
    renderIssues: diag.renderIssues,
    pageState: {
      readyState: document.readyState,
      title: document.title,
      bodyChildren: document.body?.children.length || 0,
      bodyVisible: document.body ? window.getComputedStyle(document.body).display !== 'none' : false,
      hasReactRoot: !!document.getElementById('root'),
      hasScripts: document.querySelectorAll('script').length,
      hasStyles: document.querySelectorAll('link[rel="stylesheet"], style').length
    }
  }, null, 2);
})();
```

#### Step B: Analyze & Prioritize

Group errors by category:
1. **Critical** (P0): Page doesn't render at all
   - No body, white screen, uncaught syntax errors
   
2. **High** (P1): Major functionality broken
   - Component crashes, undefined functions, missing imports
   
3. **Medium** (P2): Features degraded
   - Failed API calls, missing resources, React warnings
   
4. **Low** (P3): Cosmetic issues
   - Console warnings, deprecation notices

Identify the ROOT error (the first one that triggers others).

#### Step C: Generate Fixes

For each error:

1. **Parse the error message** to understand:
   - What file has the issue?
   - What line number?
   - What's the actual problem?

2. **Read the problematic file**
```
   !cat src/components/YourComponent.jsx
```

3. **Determine the fix**:
   - Missing import? Add it
   - Typo in variable name? Correct it
   - Wrong path? Fix the path
   - Undefined prop? Add default or prop validation
   - Syntax error? Fix the syntax

4. **Apply the fix** using Write tool

5. **Document the change**
```
   Fixed: [Error message]
   File: [Filename]
   Change: [What was changed]
   Reason: [Why this fixes it]
```

#### Step D: Reload & Verify
```javascript
// Clear logs
if (window.__diagnostics) {
  window.__diagnostics.errors = [];
  window.__diagnostics.warnings = [];
  window.__diagnostics.networkErrors = [];
}

// Hard reload
window.location.reload(true);
```

Wait 2 seconds for page to load completely.

#### Step E: Evaluate Progress

Compare error counts:
- **Better**: Error count decreased → Continue to next iteration
- **Same**: No change → Try alternative fix approach
- **Worse**: More errors → Revert last change (if possible)
- **Clean**: Zero critical errors → Success!

### 📈 Final Summary
```
╔═══════════════════════════════════════════════════════╗
║           BROWSER FIX SESSION COMPLETE                ║
╠═══════════════════════════════════════════════════════╣
║ URL: http://localhost:8080                            ║
║ Iterations: X/5                                        ║
║ Duration: Xm Ys                                        ║
╠═══════════════════════════════════════════════════════╣
║ BEFORE → AFTER                                         ║
╠═══════════════════════════════════════════════════════╣
║ Critical Errors:  X → Y  (Z% reduction)               ║
║ Total Errors:     X → Y  (Z% reduction)               ║
║ Warnings:         X → Y  (Z% reduction)               ║
║ Network Issues:   X → Y  (Z% reduction)               ║
╠═══════════════════════════════════════════════════════╣
║ FIXES APPLIED                                          ║
╠═══════════════════════════════════════════════════════╣
║ 1. [Filename]                                          ║
║    ✓ Fixed: [Description]                             ║
║                                                        ║
║ 2. [Filename]                                          ║
║    ✓ Fixed: [Description]                             ║
╠═══════════════════════════════════════════════════════╣
║ REMAINING ISSUES                                       ║
╠═══════════════════════════════════════════════════════╣
║ [If any issues remain, list them here with]           ║
║ [recommendations for manual fixes]                     ║
╠═══════════════════════════════════════════════════════╣
║ STATUS: [✅ Clean | ⚠️ Mostly Fixed | 🛑 Issues Remain] ║
╚═══════════════════════════════════════════════════════╝

Next Steps:
- [Recommendations if needed]
- Run `git diff` to review all changes
- Test user flows to ensure functionality
```

---

## Command Usage
```bash
/fix-browser-crash
```

That's it! The command will:
1. ✅ Open localhost:8080
2. 🔍 Install error monitoring
3. 🔄 Iteratively fix issues (up to 5 rounds)
4. 📊 Report results

---

## What Gets Fixed Automatically

| Issue Type | Examples | Success Rate |
|------------|----------|--------------|
| Import errors | Missing imports, wrong paths | ~95% |
| Syntax errors | Typos, missing brackets | ~90% |
| Undefined refs | Variables, functions | ~85% |
| Resource loading | CSS, JS, images | ~80% |
| React errors | Component issues | ~75% |
| Type errors | Prop types, conversions | ~70% |
| API issues | Endpoints, CORS | ~60% |

---

## Prerequisites

- ✅ Chrome browser running
- ✅ Control Chrome MCP connected
- ✅ Dev server at localhost:8080
- ✅ Source files editable

## Pro Tips

💡 **Before running**: Commit your work (`git commit -am "Pre-fix checkpoint"`)
💡 **After running**: Review changes (`git diff`)
💡 **If stuck**: Command will explain what it couldn't fix
💡 **Complex apps**: May need multiple runs for cascading fixes

