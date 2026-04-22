---
name: start-task
description: Bootstrap a new task session with plan context, 
  previous task summary, and current git state.
---
# Start Task

The user wants to begin a new task from the task plan.
Use $ARGUMENTS to identify the task (e.g. "/start-task 3" 
means Task 3).

## Your workflow:

1. **Load the plan preamble + only the requested task block.**
   `/start-task` must not read sibling tasks — this would pollute
   context and break task isolation. The plan brings three
   distinct pieces of context, and only two apply now: the
   preamble (global rules) and the requested task block. Sibling
   tasks are explicitly out of scope.
   - Locate the plan file: `docs/plans/*.md` or `plan.md` in the
     project root. If no task number is given in $ARGUMENTS,
     ask which task to start.
   - Preamble (everything before the first task heading):
     `awk '/^## Task [0-9]/{exit} {print}' <plan-file>`
   - Task block only (N is the requested task number). Note the
     `f==0` instead of `!f` — a leading `!` in an interactive
     bash triggers history expansion and breaks the command:
     `awk -v n=N 'BEGIN{f=0} f==0 && $0 ~ "^## Task " n "($|[: ])" {f=1; print; next} f && /^## Task [0-9]/ {exit} f' <plan-file>`
   - Do NOT read the spec (`docs/specs/`). If the task block
     references the spec or a sibling task, flag this back to
     the user before proceeding — the plan violates `/plan`'s
     self-containment rule and should be amended first.

2. **Read the previous task summary.** Look in:
   - `docs/task-log/` for the most recent summary file
   - Use `ls -t docs/task-log/ | head -1` to find it
   - If this is Task 1, skip this step

3. **Check git state:**
   - `git log --oneline -5` — recent commits
   - `git status --short` — any uncommitted changes
   - `git diff --stat HEAD~1` — what changed last

4. **Produce a Task Briefing** with this structure:

   ### Task N: <title from plan>
   
   **Scope:** What this task should accomplish (from plan)
   
   **Context from previous task:**
   - Key decisions made
   - Files modified
   - Open issues carried forward
   
   **Current repo state:**
   - Recent commits
   - Uncommitted changes (if any)
   
   **Proposed approach:**
   1) Assumptions
   2) Risks
   3) File-level plan
   4) Testable artifact & review guard for the next task
   5) Test plan
   6) Rollback plan
   
   Do not write code yet.

   On point (4): what concrete output proves this task worked,
   and what can the next task treat as "validated"? If there is
   no clear answer, stop and flag back to the user — the task
   is likely too large or too vague. This gate also catches
   oversized tasks from plans written outside `/plan`.

5. **Wait for user approval** before proceeding.
