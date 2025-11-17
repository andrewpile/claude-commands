`/analyze-bug-report`
Analyze a bug report: summarize the issue, extract reproduction steps, identify likely root causes, suggest prioritized fixes, and propose tests and labels.

`/analyze-pr-comments`
A prompt/template for analyzing pull request review comments: summarize feedback, classify comments (blocking, nit, suggestion), produce an action checklist, and draft suggested responses or edits.

`/catchup`
A prompt to generate a concise “catch-up” summary for a branch, issue, or PR: lists recent activity, .plan/ files, key decisions, open questions, and suggested next steps for someone returning to the work. Primes LLM for future questions.

`/fix-browser-crash`
A troubleshooting template for browser crashes: steps to reproduce, information to collect (logs, stack traces, environment), possible causes, and recommended debugging/fix strategies.

`/merge-main`
A step-by-step template for merging main into a branch (or vice versa): recommended git commands, conflict-resolution tips, and pre-merge checks (tests, CI green, changelog).

`/npm-upgrade`
A workflow template to upgrade npm dependencies safely: steps for updating package.json, running tests, regenerating lockfiles, running audits, and preparing a dependency-upgrade PR.
