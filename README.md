# Skill Kit: Agentic Workflow Skills

Fünf Claude-Code-Skills für einen reproduzierbaren Agentic-Coding-Workflow: **Plan → Start-Task → Wrap-Up**, plus `/handoff` für Context-Engpässe und `/review` als Qualitäts-Check. Gedacht für Senior Developers, die mit Claude Code in Legacy-Enterprise-Codebasen arbeiten und ihr Senior Judgment behalten wollen — nicht wegautomatisieren. Installation: entweder per `git clone` + Copy, oder einfach _„Claude, lies die `README.md` in diesem Repo und leg die fünf Skills projekt-lokal an."_ (Details unten.)

---

## Pattern, keine Vorgabe

Das hier ist **ein** Weg, nicht *der* Weg. Die fünf Skills sind die Form, in der *mein* Agentic Workflow am zuverlässigsten läuft — ich habe gute Erfahrungen damit gemacht, deshalb sind sie hier aufgeschrieben. Niemand ist darauf angewiesen, genau diese Skills in genau dieser Form zu übernehmen — auch nicht meine.

Skills sind billig und trivial änderbar: eine Markdown-Datei, ein paar Zeilen YAML-Frontmatter, Aufruf über Slash-Command. Daraus folgt: **Bau dir deine eigenen**, entlang deiner eigenen Best Practices. Wer zum Beispiel streng nach Test-Driven Development arbeitet, modelliert Red/Green/Refactor als eigene Skills. Wer feste Release-Rituale hat, baut dafür einen. Wer in Pair-Programming-Mustern denkt, formalisiert die dort. Die Taktung und die Trigger-Wörter gehören dir.

**Eine einzige Leitplanke:** Nicht zu viel automatisieren. Jeder Skill, der Entscheidungen ohne den Menschen trifft, nimmt Senior Judgment raus — und genau das ist die Substanz, die hier geschützt werden soll. Skills sollen Vorarbeit leisten (Kontext einsammeln, Templates vorbereiten, Artefakte ablegen), nicht Urteile fällen. Der Mensch bleibt in der Mitte, egal wie das Skill-Set zugeschnitten ist.

---

## Verzeichnisstruktur im Überblick

Das vollständige Bild, bevor es in die Installation geht — was liegt wo, und welcher Skill produziert welche Datei?

```
project-root/
├── .claude/
│   └── skills/
│       ├── plan/SKILL.md          # Spec → Task-Plan
│       ├── start-task/SKILL.md    # Task-Bootstrap
│       ├── wrap-up/SKILL.md       # Task-Ende & Re-Plan
│       ├── handoff/SKILL.md       # Context-Übergabe (Exception)
│       └── review/SKILL.md        # Review-Brief
├── docs/
│   ├── specs/                     # Intent: was wird gebaut und warum (Input für /plan)
│   │   └── <feature>.md
│   ├── plans/                     # Scope: Task-Reihenfolge (Output /plan, Input /start-task)
│   │   └── <feature>.md
│   └── task-log/                  # Protokoll: was tatsächlich passiert ist (Output /wrap-up)
│       └── task-{N}-{YYYY-MM-DD}-{slug}.md
├── handoff.md                     # Temporär, gitignored (Output /handoff)
└── .gitignore
```

**Drei Schichten, drei Lebenszyklen:**

- **`.claude/`** — Tool-Config, an Claude Code gebunden. Beim Tool-Wechsel wandert das nicht mit.
- **`docs/`** — Prozess-Gedächtnis, tool-agnostisch, versioniert, teilbar. Das Herz des Workflows.
- **`handoff.md`** — ephemer. Entsteht nur bei Context-Not, wird nach Erfolg gelöscht, bleibt aus dem Repo raus.

**Datenfluss zwischen den Skills:**

`docs/specs/X.md` → `/plan` → `docs/plans/X.md` → `/start-task N` → Implementierung → `/wrap-up` → `docs/task-log/task-N-...md` + Commit.

`/handoff` ist der Seitenausstieg bei Context-Knappheit (schreibt `handoff.md`), `/review` der Qualitäts-Check zwischendurch oder vor PR (schreibt nichts, produziert einen Brief an den Menschen).

---

## Installation

Jeder Skill ist eine `SKILL.md`-Datei in einem eigenen Unterordner. Das Repo liefert beides: eine aufgesplittete Form unter [`skills/`](./skills/) — und die eingebetteten Versionen weiter unten in dieser README, damit Claude sie aus einer einzigen Datei anlegen kann.

### Option 1 — Claude liest die README und legt die Skills an (empfohlen)

```
"Claude, lies die README.md in diesem Repo und leg die fünf Skills projekt-lokal unter .claude/skills/ an.
Lege außerdem docs/specs/, docs/plans/, docs/task-log/ an und ergänze handoff.md in .gitignore."
```

Das funktioniert, weil die vollständigen Skill-Bodies weiter unten eingebettet sind. Für den Moment, in dem das System zum ersten Mal in einem Repo eingerichtet wird, ist das der kürzeste Weg — Claude holt sich nebenbei den Kontext, *warum* die Skills so aussehen, wie sie aussehen.

### Option 2 — Git-Clone + Copy

```bash
git clone https://github.com/<user>/skill-kit-agentic-workflow.git
cd <dein-projekt>

# Skill-Ordner (projekt-lokal, versionierbar, teambar)
mkdir -p .claude/skills
cp -r /pfad/zu/skill-kit-agentic-workflow/skills/* .claude/skills/

# Prozess-Artefakt-Ordner
mkdir -p docs/{specs,plans,task-log}

# handoff.md ist temporär — nicht versehentlich committen
echo "handoff.md" >> .gitignore

git add .claude/ docs/ .gitignore
git commit -m "chore: agentic workflow scaffold"
```

### Global vs. projekt-lokal

- **Global** (`~/.claude/skills/`) — über alle Projekte verfügbar, nicht versioniert. Gut für den Solo-Einstieg.
- **Projekt-lokal** (`.claude/skills/`) — im Repo versioniert, teambar, per Fork anpassbar. **Das ist für den Workshop und Team-Setups die bessere Wahl.**

### Trennung: Tool-Config vs. Prozess-Artefakte

Wichtig für das Mental Model:

- **`.claude/`** ist Tool-Konfiguration — Skills und `settings.json`. Bleibt klein, bleibt Claude-Code-spezifisch.
- **`docs/`** ist das Prozess-Gedächtnis des Projekts — tool-agnostisch, sprechend benannt, lesbar auch ohne Claude Code:
  - `docs/specs/` — was gebaut werden soll und warum
  - `docs/plans/` — Reihenfolge und Scope
  - `docs/task-log/` — was tatsächlich passiert ist (Task-Summaries)

Task-Summaries gehören **nicht** in `.claude/` — sie sind Projekt-Dokumentation, keine Tool-Config. Wenn das Team morgen auf Cursor oder Codex wechselt, wandert der Task-Log mit, die Skills nicht.

---

## Übersicht

| Skill         | Typ           | Phase                                              | Trigger                                                                  |
| ------------- | ------------- | -------------------------------------------------- | ------------------------------------------------------------------------ |
| `/plan`       | Routine       | Spec → Plan (vor Task-Beginn)                      | "Plan spec X" / "Schneide spec X in Tasks"                               |
| `/start-task` | Routine       | Task-Beginn                                        | "Starte Task N"                                                          |
| `/wrap-up`    | Routine       | Task-Ende (DONE oder BLOCKED)                      | "Bin fertig" / "Task blockiert, re-plan"                                 |
| `/handoff`    | **Exception** | Context knapp *vor* Task-Ende, oder bewusste Pause | "Context wird knapp, Übergabe vorbereiten" / "Ich pausiere hier manuell" |
| `/review`     | Routine       | Nach Task (quick) oder vor PR (full)               | "Review bitte" / "Review full"                                           |

**Wichtig — Routine vs. Exception:** `/wrap-up` ist der Normalabschluss jedes Tasks, `/handoff` die Ausnahme. Im Happy-Path kommt `/handoff` nie zum Einsatz. Du brauchst ihn nur, wenn (a) der Context-Verbrauch kritisch wird, bevor du den Task fertig hast, oder (b) du einen Task bewusst pausierst, um ihn manuell nachzuarbeiten. Beide Fälle sind Spezialfälle — in der Summe deutlich seltener als `/wrap-up`.

**Minimales Setup:** `/wrap-up` allein bringt schon den meisten Wert. Der Rest ist optional und addiert sich inkrementell.

**Empfohlene Reihenfolge beim Einführen:** `/wrap-up` → `/start-task` → `/plan` → `/review` → `/handoff`
*(Der Exception-Skill `/handoff` zuletzt — wer die Routine-Skills nicht nutzt, braucht die Ausnahme erst recht nicht. `/plan` kommt bewusst erst nach `/start-task`: die Task-Seed-Kette trägt bereits ohne ihn, er veredelt sie nur um das front-loaded Sizing.)*

**Bewusst kein eigener Skill:**
- **Re-Planning** ist in `/wrap-up` integriert. Wenn ein Task BLOCKED endet, liefert `/wrap-up` zusätzlich eine Escalation-Bewertung und einen Re-Plan-Vorschlag. Ein separater `/replan`-Skill wäre doppelt gemoppelt — ein BLOCKED-Task ohne Re-Plan-Gedanken ist eh sinnlos.
- **Einfaches Debugging** (klarer Stacktrace, reproduzierbar) läuft über die normale Task-Tool-Delegation an einen Subagenten — kein Skill nötig.
- **Exploratives Debugging / Isolations-Experimente** (eigener Nebenkriegsschauplatz, der später zurückgekoppelt werden soll) passiert auf Tool-Ebene, nicht auf Skill-Ebene. Siehe Abschnitt *Isolations-Experimente — ohne eigenen Skill* weiter unten — die Entscheidung zwischen neuer Instanz, `/handoff`, `/branch`, Rewind und ephemerer Frage ist ein Senior-Call.

### /wrap-up vs. /handoff — wann was?

|               | `/wrap-up`                                                   | `/handoff`                                                   |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Situation** | Task ist fertig (oder blockiert)                             | Task ist nicht fertig, aber Context wird knapp               |
| **Ergebnis**  | Task-Summary im Repo, gemeinsam mit Code committet           | Temporäre `handoff.md` für die nächste Session               |
| **Danach**    | Nächster Task                                                | Gleicher Task fortsetzen in frischer Session                 |
| **Datei**     | `docs/task-log/task-{N}-{YYYY-MM-DD}-{slug}.md` (persistent) | `handoff.md` im Projekt-Root (temporär, nach Erfolg löschen) |
| **Analogie**  | Schichtprotokoll                                             | Staffelstab                                                  |

---

## 1. /plan

Nimmt eine Spec (aus `docs/specs/`, Chat oder Issue) und schlägt einen passend geschnittenen Task-Plan für `docs/plans/` vor. Die Slicing-Regeln sind so gewählt, dass Plan-Entry und späterer `/wrap-up`-Commit denselben Standard teilen — *ein Task = ein kohärenter, testbarer Commit*.

**Datei:** `.claude/skills/plan/SKILL.md` · [im Repo ansehen](./skills/plan/SKILL.md)

````markdown
---
name: plan
description: Break a spec into a right-sized, testable task plan.
  Produces a proposal; writes to docs/plans/ only after user approval.
  Each task is sized so that its diff + test could land as a single commit.
---
# Plan a Spec

The user has a specification (in `docs/specs/`, a chat message, or
a linked issue) and wants to turn it into an ordered task list that
`/start-task` can consume.

Use `$ARGUMENTS` to locate the spec (e.g. `/plan docs/specs/auth-rewrite.md`).
If no argument is given, ask which spec to plan.

## Workflow

1. **Read the spec in full.** Do not plan from a summary — read the
   whole document so acceptance criteria and edge cases are visible.

2. **Check repo state:**
   - `git log --oneline -10`
   - `ls docs/task-log/ 2>/dev/null` — is there related recent work?
   - `ls docs/plans/ 2>/dev/null` — does a plan already exist for this spec?

3. **Propose a task breakdown** following the rules below.
   Present it to the user **before** writing any file.

4. **Wait for user approval.** Only then write to
   `docs/plans/{slug}.md`. If a plan file already exists, propose
   an edit rather than overwriting — the user decides.

## Task-sizing rules (apply to EACH task)

- **One coherent goal.** No multi-goal items joined by "and" or
  "then". If those words appear in the title, it's two tasks.
- **Independently reviewable.** The task's diff + its test could
  land as a single commit with no cross-task coordination.
- **Self-contained vs. the spec.** The task must be interpretable
  and executable without re-reading the spec. `/start-task` does
  not load `docs/specs/`. Inline every spec-derived fact the
  executing agent needs — constraints, acceptance criteria, domain
  rules, invariants — into `Instructions`, `Acceptance`, or
  `Key Discoveries`. A task that says "implement section 3.2 of
  the spec" is broken by definition.
- **Acceptance built in.** Each task carries its own verification —
  prefer automated tests (unit or integration). If automation is
  a bad fit, acceptance may be omitted rather than prescribing
  manual steps. For pure mechanical refactors with no behavior
  change, acceptance is optional (existing tests staying green
  is implicit).
- **No standalone "testing", "verification", or "stabilization"
  tasks.** Tests belong to the task that introduces the behavior.
  A separate test-task is a size smell — the behavior-task was
  too large, or the test-task lacks a reviewable outcome of its own.
- **At most one explicit dependency** on a previous task. Deep
  dependency chains mean the slicing is wrong.
- **Size target (orientation, not rule):** completable in one
  sitting, rarely more than a few hours and ~10 files. Legacy-heavy
  touchpoints may exceed this — flag it when they do.

## Slicing rubric

- **TOO LARGE** if it spans multiple subsystems, sweeping refactors,
  or has ambiguous outcomes — split by subsystem or by
  "behavior change" vs "refactor".
- **TOO SMALL** if it lacks a distinct, reviewable outcome or test —
  merge into its parent goal.
- **JUST RIGHT** if the diff + test could be reviewed and landed
  as a single commit. Related changes within one subsystem
  (including tests) stay in the same task.

Aim for a handful of tasks. If the count climbs past ~8, the
slicing is probably too fine or the spec is too ambitious for one
plan — propose splitting the spec instead.

## Task entry schema

For each task, produce:

- **Title** — short, imperative, unambiguous.
- **Instructions** — what to do (Markdown encouraged). Must be
  self-contained: no references like "see spec" or "as in Task 2".
  The executing agent will read only this block plus the plan
  preamble.
- **Acceptance** — how to verify success. Optional only for pure
  mechanical refactors.
- **Key Locations** — files, fully qualified classes/methods
  likely to be touched.
- **Key Discoveries** — facts from the spec or outside the key
  locations that the executing agent needs at start (domain rules,
  invariants, acceptance nuances). Required whenever the spec
  contains such facts; omit only for trivial refactors where spec
  and task scope are effectively identical.

## Plan file layout

The plan file uses a stable heading convention so `/start-task N`
can extract exactly one task block without loading siblings:

- **Preamble** — everything before the first task heading: plan
  title, spec reference, scope note, and the Flexibility Clause.
  Keep it compact (≤ 30 lines). This is the only cross-cutting
  context `/start-task` will load alongside the requested task.
- **Task heading** — `## Task N` or `## Task N: <title>` on its
  own line. Tasks are numbered sequentially starting at 1.
- **Task block** — from `## Task N` up to (but excluding) the
  next `## Task M` heading or EOF. `/start-task N` extracts
  exactly this block.

Do not nest tasks under deeper headings (`### Task N`) and do
not split a task across multiple `## Task N` occurrences — the
extraction pattern anchors on `^## Task <number>`.

## Flexibility clause (include verbatim in the plan)

> The executing agent may adjust scope and ordering based on more
> up-to-date context discovered during implementation, as long as
> each task still satisfies the sizing rules above.

This keeps the plan a guide, not a straitjacket.

## After user approval

1. Write the plan to `docs/plans/{slug}.md`. Create `docs/plans/`
   if it does not exist.
2. Do NOT commit automatically. Tell the user:
   > Plan is ready at `docs/plans/{slug}.md`.
   > Recommended commit:
   > ```bash
   > git add docs/plans/<file>
   > git commit -m "plan: <spec title>"
   > ```
3. The plan now feeds `/start-task N` for execution.
````

### Warum funktioniert das?

Das Sizing-Kriterium hier ist dasselbe, das `/wrap-up` später durchsetzt: *ein Task = ein zusammenhängender, getesteter Commit*. Plan-Entry und Commit-SHA beschreiben dieselbe Einheit, nur an unterschiedlichen Punkten in der Zeit. Das hat drei praktische Effekte:

- **Schlechtes Sizing fällt beim Schreiben auf.** Wer die Acceptance leer lassen muss, merkt sofort: der Task hat keinen testbaren Kern — zu groß oder zu vage.
- **Jeder nächste Task startet auf validiertem Boden.** Jeder Baustein trägt seinen eigenen Beweis; Folgeschritte bauen auf etablierten, nicht bloß vermuteten Zuständen auf. Die Acceptance wird zum Review-Guard für den darauffolgenden Task.
- **„Testing am Ende" fällt aus.** Das Anti-Pattern „erst baue ich vier Dinge, dann mache ich einen Test-Task drauf" wird durch die Regel *keine Standalone-Test-Tasks* explizit verboten — Tests leben dort, wo sie etwas validieren, nicht in einer eigenen Aufräum-Phase.

Das Muster ist in vergleichbarer Form in Produktions-Agenten (z.B. Brokks `createOrReplaceTaskList`) operationalisiert. Die harten Regeln — ein Ziel pro Task, keine Test-only-Tasks, Diff + Test als ein Commit — haben sich dort bewährt. Die Skill-Version übernimmt den Kern und lässt Brokks Tool-Call-Mechanik (Verbatim-Copy bei Inkremental-Updates, `List<TaskListEntry>`-Strukturierung) weg, die nur in einer Agent-UI sinnvoll ist.

---

## 2. /start-task

Bootstrapped eine neue Task-Session mit dem richtigen Kontext. Verhindert, dass du den Seed-Prompt jedes Mal manuell tippen musst.

**Datei:** `.claude/skills/start-task/SKILL.md` · [im Repo ansehen](./skills/start-task/SKILL.md)

```markdown
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
```

---

## 3. /wrap-up

Generiert eine strukturierte Task-Summary und committet sie. Das Kern-Skill des gesamten Workflows — sorgt für die Seed-Kette zwischen Tasks.

**Datei:** `.claude/skills/wrap-up/SKILL.md` · [im Repo ansehen](./skills/wrap-up/SKILL.md)

```markdown
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
```

---

## 4. /handoff

> **Exception-Skill, kein Routine-Baustein.** Der Normalabschluss eines Tasks ist `/wrap-up`. `/handoff` ist nur für zwei Fälle gedacht: (a) Context wird knapp *bevor* der Task fertig ist, oder (b) du pausierst den Task bewusst, weil manuelle Nacharbeit ansteht und du später in frischer Session weitermachst.

Extrahiert den relevanten Kontext aus einer laufenden Session in ein temporäres Übergabe-Dokument. Die neue Session liest die `handoff.md` und setzt die Arbeit ohne Explorations-Overhead fort.

**Datei:** `.claude/skills/handoff/SKILL.md` · [im Repo ansehen](./skills/handoff/SKILL.md)

```markdown
---
name: handoff
description: Extract relevant context from the current 
  session into a temporary handoff document for seamless 
  continuation in a fresh session.
---
# Session Handoff

The user's context window is filling up (typically 40-50%).
The current task is NOT finished — analysis is done but 
implementation remains. Create a handoff document that 
enables a fresh session to continue without re-exploration.

## Create `handoff.md` in the project root.

### Document structure:

#### Background
2-3 sentences: What is the task? What is the goal?
No history, no conversation recap — just the objective.

#### Already Done
What was already implemented in THIS session or earlier.
Include concrete code references (file, method, pattern).
This serves as a **few-shot example** for the remaining 
work — the next session will use it to understand the 
expected solution style.

#### TODO
For each remaining fix/change:

1. **Title** — one-line description
2. **File and method** — exact location with line numbers
3. **Current code** — the actual snippet that needs changing
4. **Problem** — why it's wrong (one sentence)
5. **Proposed fix** — concrete code or pseudocode
6. **Complexity** — Very low / Low / Medium / High
7. **Risk** — None / Performance / Breaking change / etc.

Be specific. Vague TODOs waste context in the next session.

#### Key Files to Read
Table format:
| File | Why |
|------|-----|
Each entry: file path + one sentence explaining what 
the next session needs from this file.

Only include files that are NECESSARY for the remaining 
work. Not everything that was read — only what matters.

#### Architecture Context
If the task requires understanding non-obvious 
relationships between components, include a brief 
sketch. Example:


MultiAnalyzer
├── delegates: Map<Language, IAnalyzer>
└── templateAnalyzers: Collection<ITemplateAnalyzer>


Only include this if it saves the next session from 
having to rediscover the architecture.

#### Complexity Summary
Table format:
| Fix | Complexity | Risk |
|-----|------------|------|

### What NOT to include:
- Conversation history
- Rejected approaches or dead ends
- Files that were read but aren't needed for the fix
- Broad project context that's already in CLAUDE.md

### Guiding principle:
**Maximum signal density.** Every line in handoff.md 
should save the next session from doing redundant work.
If a line doesn't directly help with implementation, 
cut it.

## After creating handoff.md:

1. Tell the user: "handoff.md is ready. In the new 
   session: `/clear` → then say 'Read handoff.md 
   and follow the instructions.'"
2. Do NOT commit handoff.md — it is temporary.
3. Remind the user to delete handoff.md after the 
   task is completed successfully.
```

### Warum funktioniert das?

Ein Praxisbeispiel aus einer realen Brokk-Session — drei Fixes in `ContextAgent.java` für Angular Template Support:

```
Session 1 (Analyse)                    Session 2 (Implementierung)
┌─────────────────────┐                ┌─────────────────────┐
│ Exploration         │                │ /clear              │
│ Architektur-Analyse │                │ "Lese handoff.md"   │
│ Fix-Strategie       │                │ → Sofort produktiv  │
│                     │                │ → Alle 3 Fixes      │
│ Context: 50% ──────►│── handoff.md ─►│ → BUILD SUCCESSFUL  │
│ (nur Analyse!)      │                │                     │
│                     │                │ Context: 28%        │
│                     │                │ (Analyse + Impl!)   │
└─────────────────────┘                └─────────────────────┘
```

Die handoff.md hatte 154 Zeilen und enthielt:

- **Already Done** — ein bereits implementierter Fix als Referenz-Pattern (implizites Few-Shot)
- **3× TODO** — jeweils mit aktuellem Code, Problem, konkretem Fix-Vorschlag
- **Leseliste** — 6 Dateien mit Begründung, warum jede gelesen werden muss
- **Architektur-Skizze** — `MultiAnalyzer → delegates vs. templateAnalyzers`
- **Komplexitätstabelle** — Fix A: trivial, Fix B: mittel (Performance-Risiko), Fix C: niedrig

Die neue Session las die handoff.md, arbeitete die Leseliste zielgerichtet ab (statt zu suchen), verwendete den Already-Done-Fix als Lösungsmuster, und implementierte alle drei Fixes in 3 Minuten — mit BUILD SUCCESSFUL beim ersten Versuch.

**Context ist ein Budget. Die handoff.md ist Budgetplanung.**

### Industrievalidierung

Sourcegraph Amp hat Session Handoff als **eingebautes Feature** mit eigenem Slash Command. Der Agent erkennt automatisch, wann der Context knapp wird, und bietet die Erstellung eines Handoff-Dokuments an. Das Pattern ist so fundamental, dass professionelle Coding-Agents es als First-Class-Feature behandeln.

---

## 5. /review

Generiert einen Guided Review Brief mit Hotspots, Cross-Task Concerns und Blind Spots.

**Datei:** `.claude/skills/review/SKILL.md` · [im Repo ansehen](./skills/review/SKILL.md)

```markdown
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
```

---

## Isolations-Experimente — ohne eigenen Skill

Manchmal will man einen Nebenkriegsschauplatz aufmachen: einen Bug isoliert reproduzieren, eine Alternative durchspielen, eine Frage stellen, deren Antwort den Hauptkontext nicht belasten soll. Ursprünglich war dafür ein `/debug-fork`-Skill gedacht — er ist bewusst wieder rausgeflogen. Der Grund: **Isolation passiert auf Tool-Ebene, nicht auf Skill-Ebene.** Ein Skill kann dem Agenten nur sagen, er möge sich isoliert verhalten — er kann die Session nicht tatsächlich forken. Die eigentliche Isolation muss Claude Code selbst liefern, und dafür gibt es mehrere native Mechanismen mit unterschiedlichen Kontext-Semantiken.

**Die fünf Varianten im Überblick:**

- **Neue Agenteninstanz** — startet kontextlos. Passt für Fragen, bei denen der aktuelle Kontext keinen Mehrwert hätte.
- **`/handoff` → neue Session** — bewusster, kuratierter Kontext-Transfer. Passt, wenn der Ausflug handfeste Artefakte produziert (Bugfix, Prototyp), die manuell in den Haupt-Ast zurückgemeldet werden.
- **`/branch`** (Claude Code nativ) — zweigt die Session mit vollem Kontext ab, der Haupt-Ast bleibt unberührt. Meistens die richtige Wahl, wenn man zurückkoppeln will, ohne vorher manuell den Kontext auswählen zu müssen.
- **Frage + Rewind** — Frage stellen, Antwort lesen, Konversation zurückdrehen. Passt für kurze Recherche-Fragen, deren Antwort den Hauptkontext nicht aufblähen soll. (Achtung: Rewind spult die Konversation zurück, **nicht die Dateiänderungen** — wenn der Agent während der Frage etwas geschrieben hat, bleibt das auf der Platte.)
- **Ephemere Frage** — voller Kontext verfügbar, aber die Antwort landet nicht im Konversations-Kontext. Passt für schnelle Nachfragen, deren Antwort man nur kurz braucht, ohne den Folgeverlauf zu vergiften.

**Leitfragen für die Wahl:**

1. *Wie viel Kontext brauche ich am Start?* — leer, voll, oder kuratiert?
2. *Will ich das Ergebnis zurückkoppeln?* — wenn ja: `/branch` oder `/handoff`. Wenn nein: ephemere Frage, Rewind oder neue Instanz.

Keine dieser Varianten lohnt einen eigenen Skill — sie sind Tool-Features und jede Automatisierung der Wahl zwischen ihnen würde genau die Senior-Entscheidung wegnehmen, die der Skill-Kit-Ansatz ausdrücklich beim Menschen behalten will.

---

## Anpassung und Erweiterung

Die Skills sind bewusst als Instruktionen formuliert, nicht als Code. Das heißt: du kannst sie jederzeit anpassen, ohne etwas zu kompilieren oder zu deployen. Einfach die `SKILL.md` editieren.

**Typische Anpassungen:**

- Summary-Format ändern (z.B. andere Felder, anderes Schema)
- Review-Kriterien projektspezifisch erweitern (z.B. Security-Checks für Fintech)
- Escalation-Trigger anpassen (z.B. strengere Schwellen für regulierte Codebases)
- Commit-Message-Prefixes an Team-Konventionen anpassen
- Handoff-Struktur erweitern (z.B. mit Dependency-Graph oder API-Contracts)

**Eigene Skills bauen:**

Das Muster ist immer gleich: YAML-Frontmatter mit `name` und `description`, dann Markdown mit Instruktionen. `$ARGUMENTS` enthält alles was nach dem Slash-Command getippt wird (z.B. `/start-task 3` → `$ARGUMENTS = "3"`).

```
.claude/skills/<name>/SKILL.md
```

Der `name` im Frontmatter wird automatisch zum `/slash-command`.

---

## Lizenz & Beiträge

MIT — siehe [LICENSE](./LICENSE). Fork, passe an, teile zurück. Issues und Discussions sind der richtige Ort für Feedback aus der Praxis.
