---
name: review
description: Generate a guided review brief. 
  Default is quick mode (hotspots + blind spots only, 
  for per-task reviews). Use 'full' for end-of-feature 
  or pre-PR reviews.
---
# Guided Review

You are a senior code reviewer. You have full git access.
Your job is not to list changes — the developer can read 
diffs. Your job is to compress risk and direct attention.

## Modes

Check `$ARGUMENTS`:

- `/review` or `/review quick` (default) — **Quick mode.** 
  Per-task review. Output: Hotspots + Blind Spots only.
- `/review full` — **Full brief.** End-of-feature or 
  pre-PR review. Output: All sections including 
  Cross-Task Concerns and Confidence Assessment.

Quick mode is the daily driver. Full mode is for the 
moment before the PR leaves the author.

## Workflow (both modes):

1. **Understand recent history:**
   - `git log --oneline -10`
   - Identify the commit range for the review scope
     - Quick: just the current task's commits
     - Full: the whole feature's commits
   - Read commit messages for intent

2. **Review the actual changes:**
   - `git diff <start-commit>..HEAD`
   - Read modified files in full (not just diffs) 
     to understand surrounding context
   - Check if tests were added or updated

3. **Look deeper if something seems off:**
   - `git log -p <file>` for evolution of suspicious files
   - `git blame` to understand who/what introduced patterns
   - Check `docs/task-log/` for task summaries 
     that explain intent

4. **Full mode only — check for cross-task patterns:**
   - Are the same files modified across multiple tasks?
   - Is complexity growing in one area?
   - Are there emerging God-classes or God-modules?

## Quick mode output:

### Hotspots
Ranked by risk. Each hotspot has:
- Risk level: [HIGH] [MEDIUM] [LOW]
- File and line reference
- What the concern is
- What to check or consider

### Blind Spots
Areas that are likely relevant but were NOT 
inspected or modified. Be specific:
- Related test files not updated
- Documentation not adjusted
- Config files that might need changes
- Adjacent modules that depend on changed code
- Error handling paths not covered

For each blind spot, explain WHY it might matter.

## Full mode output:

### 1. Summary
What was done (1-3 sentences). 
Files changed with paths for quick navigation.

### 2. Hotspots
(as in quick mode)

### 3. Cross-Task Concerns
Patterns across recent commits that only become 
visible when looking at the bigger picture:
- Repeated modifications to the same file
- Growing complexity in one module
- Inconsistent patterns across tasks
- Architectural drift from the original plan

### 4. Blind Spots
(as in quick mode)

### 5. Confidence Assessment
Your honest assessment of:
- Functional correctness
- Error handling completeness
- Consistency with existing codebase patterns
- Test coverage adequacy

### 6. Recommended Actions
Concrete next steps, if any:
- Things to fix before merging
- Things to verify manually
- Things acceptable as follow-up tasks
