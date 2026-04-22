---
name: wrap-up
description: Generate a structured task summary 
  at the end of a completed or blocked task. 
  For BLOCKED tasks, also evaluates escalation 
  triggers and proposes a re-plan.
---
# Task Wrap-Up

The task is finished (or has been declared blocked).
Create a summary file at
`docs/task-log/task-{N}-{YYYY-MM-DD}-{slug}.md`

- `{N}` = task number from the plan
- `{YYYY-MM-DD}` = today's date (use `date +%Y-%m-%d`)
- `{slug}` = short kebab-case description (e.g. `login-service`)

If the `docs/task-log/` directory does not exist, 
create it first.

If the task is NOT finished but the context is 
filling up, use `/handoff` instead — this skill is 
for completed or blocked tasks only.

## Base summary structure (always):

### Task
One-sentence summary of what was worked on.

### Status
DONE | BLOCKED
If BLOCKED, explain why and what needs to happen
to unblock. (For in-progress tasks, use `/handoff`.)

### Files Modified
Each file with a one-line description of what 
changed and why. Format:
- `path/to/file.ts` (new|modified|deleted) — reason

### Files Read (Context Only)
Files that were read for understanding but NOT 
modified. This helps the next session know what 
context was used.

### Key Decisions
Technical decisions made during this session and 
the reasoning behind them. Include alternatives 
that were considered and rejected.

### Test Evidence
What was tested and how. Include:
- Test commands run and their output
- Manual verification steps taken
- Screenshots or log snippets if relevant

### Open Issues
Unfinished work, known issues, open questions.
Reference follow-up tasks where applicable.
Format: "Issue description (→ Task N)" 

### Context for Next Task
What the next session needs to know to continue.
Include:
- Key interfaces and their signatures
- State assumptions
- Dependencies between this task and the next
- Any gotchas discovered during implementation

### Git State
Run these commands and include their output:
- `git diff --stat`
- `git status --short`

## Additional sections for BLOCKED tasks only:

If Status is BLOCKED, also evaluate the escalation 
triggers and produce a re-plan proposal. Append the 
following sections to the summary:

### Escalation Assessment
For each trigger, state: CLEAR | WARNING | TRIGGERED
with evidence.

1. **Scope creep:** `git diff --stat` — are more files 
   modified than the plan specified? Which ones are 
   outside planned scope?
2. **API changes:** Has any public interface changed 
   that was not planned? Check exports, function 
   signatures, shared types.
3. **Failed attempts:** How many implementation 
   attempts? Check git log for reverts or repeated 
   changes to the same files.
4. **Test failures outside scope:** Are tests failing 
   in modules not targeted by this task? 
   Run the test suite and check.
5. **Explainability:** Can you summarize what this 
   task accomplished in 5 bullet points or fewer? 
   If not, scope is probably too large.

### Re-Plan Proposal
Based on the assessment, propose:
- What to keep from the current task (already valuable)
- How to split the remaining work into smaller tasks
- Updated task list with revised scope

Then propose an updated plan patch for `docs/plans/`.
Do NOT overwrite the old plan — the old version stays 
in git history. Write the revision as an edit.

## After generating the summary:

Do **not** commit automatically. The user reviews 
the diff and commits the summary together with the 
task's code changes in a single commit.

Tell the user:
> Summary is ready at `docs/task-log/<filename>.md`.
> Recommended commit (code + summary together):
> 
> ```bash
> git add docs/task-log/<filename>.md <changed-source-files>
> git commit -m "task-N: <one-line summary>"
> ```
> 
> For BLOCKED with re-plan:
> ```bash
> git add docs/task-log/<filename>.md docs/plans/<plan-file> <any-code-kept>
> git commit -m "replan: <reason> (task-N blocked)"
> ```
> 
> Before committing, run `git status` and 
> `git diff --cached` to verify only the intended 
> changes are staged — no `git add -A`.

The single-commit rule matters: the summary 
describes *this exact code state*. Code and summary 
should always live in the same commit so `git log` 
shows one coherent story per task.
