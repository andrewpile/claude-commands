---
description: Merge main branch into current branch, auto-resolving conflicts when possible
allowed-tools:
  - Bash(git *)
  - Read
  - Write
---

# Merge Main Branch

!git fetch origin main
!git merge origin/main

If merge succeeds, done.

If conflicts occur:

!git status --short

For each conflicted file:
- Read the file to see conflict markers
- If one side is clearly better (e.g., one just deleted, one added new code), auto-resolve
- If both sides have substantial changes, show the diff and ask:
```
Conflict in <file>:

<<<<<<< HEAD (your changes)
<your version>
=======
<main version>
>>>>>>> origin/main

Choose:
1. Keep your changes
2. Keep main's changes  
3. Keep both (you'll need to manually edit)
```

After resolving all conflicts:

!git add .
!git commit -m "Merge main into <branch>"

Show summary of what was merged and how conflicts were resolved.
