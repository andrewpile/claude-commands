---
description: Clear history and analyze all branch changes for better context
allowed-tools:
  - Bash(git *)
  - Bash(gh *)
  - Read
---

# Catchup - Branch Context Builder

!/clear

Analyze the current branch to build context:

!git log --oneline $(git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD origin/master)..HEAD
!git diff --stat $(git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD origin/master)..HEAD
!git diff $(git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD origin/master)..HEAD

## Check for PR context

!gh pr view --json number,title,url,comments,reviews 2>/dev/null || echo "No PR found"

If PR exists, note the PR number, title, and key comments from reviewers.

## Check for planning document

Store current branch name and check for plan file:

!BRANCH=$(git rev-parse --abbrev-ref HEAD) && echo "Branch: $BRANCH"

Look for plan in .plans/ directory:
!BRANCH=$(git rev-parse --abbrev-ref HEAD) && \
  test -f ".plans/$(gh pr view --json number -q .number 2>/dev/null).md" && cat ".plans/$(gh pr view --json number -q .number).md" || \
  test -f ".plans/${BRANCH//\//-}.md" && cat ".plans/${BRANCH//\//-}.md" || \
  test -f ".plans/$(basename $BRANCH).md" && cat ".plans/$(basename $BRANCH).md" || \
  echo "No plan found"

## Summary

Now provide a summary of:
- The feature/fix being worked on
- Key modified files and their purposes  
- Overall scope and architectural patterns
- PR context (if exists): feedback, concerns, approvals
- Plan alignment (if exists): objectives, how actual work compares to plan
- Context that will help with follow-up questions
