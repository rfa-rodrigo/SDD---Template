# SDD Template

Spec-driven development in six stages: **pdr → spec → plan → tasks → execute → archive**. The written artifact is authoritative, the code is derived from it, and every stage has a human approval gate before the next one starts.

## Installation

Before opening the session, put `CLAUDE.md` at the root of the project — it holds the behavioral guidelines that apply to every session. It is loaded when the session opens, and only then: a file added later does not enter a session that is already running.

With it in place, put `SETUP-SDD.md` at the root, open the session and say:

```
Read SETUP-SDD.md at the root of this project and execute it from start to finish, following its execution rules.
```

Nothing is overwritten. When it finishes, the agent reports what it created and what it skipped — and nothing else.

The setup installs nothing, needs no network and depends on no third-party tool. It creates folders and writes files, and it skips whatever already exists instead of overwriting it.

## What the setup delivers

A skill is a file: `.claude/skills/<name>/SKILL.md`, with `name` and `description` in the header and the instructions in the body. Section 2.1 fixes that contract, and each subsection from 2.2 to 2.7 carries the literal `description` and the body of its skill. The agent copies — it does not draft, paraphrase or summarize.

The `description` is what decides when the skill fires, and all six state in writing that they fire only on explicit invocation by the user. That is the gate of the flow: a skill that fires because the subject came up starts interviewing, planning or executing without anyone having asked. That is why the text comes ready-made instead of being drafted at each installation.

```
project/
├── .claude/
│   ├── skills/          pdr, spec, plan, tasks, execute, archive
│   └── agents/
│       ├── revisor.md   audits artifacts, read-only, never fixes
│       └── executor.md  runs one task at a time, the only one that writes code
├── .spec/
│   ├── specs/           the artifacts of the feature in progress
│   │   └── archive/     one folder per finished feature
│   ├── templates/       pdr, spec, plan, tasks, exec
│   └── shared/
│       └── memory.md      what carries across features
└── resources/           the root of the code
    └── archive/         only in a project with cyclic work
```

During installation the setup asks whether the project has cyclic work: the same task repeated from time to time over different files. If it does, it creates `resources/archive/`, where the files of the cycles already done are kept: when a cycle ends, you move that cycle's files there, and `resources/` is ready for the next one. The one who moves them is always you — no skill of the flow moves or archives anything there. In a project without cyclic work, `resources/` is fixed and the folder does not exist. Not to be confused with `.spec/specs/archive/`, which keeps the artifacts of the features.

## The flow

Each stage runs, you check it, and only then does the next one start. None fires on the heels of the previous one, and none approves itself.

    /pdr        leads the conversation and records the decision    → PDR-<NNN>
    /spec       researches, proves feasibility, specifies          → SPEC-<NNN>
    /plan       reads the code and designs the engineering         → PLAN-<NNN>
    /tasks      breaks it down into atomic tasks with checkboxes   → TASKS-<NNN>
    /execute    executes, in isolation, and records what happened  → EXEC-<NNN>
    /archive    archives the artifacts and records the feature's memory

The five artifacts live in `.spec/specs/` and carry the same `<NNN>` — the feature ID, asked of the user in the PDR and checked against the active **and** archived features before being accepted. The slug follows the same path: it is born in the PDR and repeated, unchanged, by every later artifact and by the archived folder.

## The ideas that hold the flow together

**Deciding and specifying are different things.** The PDR fixes the decision and the why; the spec fixes the how. A spec that decides what the PDR did not decide is a blocking finding — that is a new PDR, not a bigger spec.

**Feasibility before proposal.** The `spec` skill researches the internet before proposing: repository, official documentation, release notes, license. Opening the source is mandatory — nothing is cited from the search result alone. For each approach it lists what has to be true for it to work and confirms item by item in a source it opened. What cannot be confirmed becomes a declared risk or brings the approach down.

**Verification is born before the code.** Every step of the plan and every task carries the command that proves it worked and the result that approves it. A step whose verification you cannot name is too big or poorly defined.

**Parallelism is derived, not wished for.** Two tasks only run together if there is no dependency between them, if they do not write to the same file and if they do not use the same resource. Writing to the same file is a dependency even without a logical dependency; and what the verification command writes — lockfile, cache, `dist/`, coverage — counts as a write. That is how you find out that running `install` in parallel was never parallel.

Derived, and still optional: the task file carries a `Paralelismo` (parallelism) field, which is born `não` (no) and only becomes `sim` (yes) if you release it in words. It governs dispatch, never analysis — the blocks keep being built and checked either way, because building the block is what proves the dependencies are right. Running in series is always safe; the field exists so that parallelizing is your choice, not the agent's default.

**Atomic is not tiny.** It is indivisible without losing value: a change that verifies on its own and would fit in one commit. Eight steps that are only worth something together were one.

**The ceremony does not pay off for everything.** The cycle is worth it for multi-session work, with invariants that cannot break, or work that will be resumed months from now. Trivial work stays out.

## Against contamination

The nightmare of the flow is the agent doing what nobody asked for. The defense has four layers, and it is literal.

**Isolation.** Every task runs in the `executor` subagent, even when it is the only one in the block. It receives the task block and nothing else: it does not see the PDR, the spec, the plan, the other tasks or the conversation. What is not written in the task does not exist for it. Whoever orchestrates does not execute.

**Literal scope statement.** Every dispatch ends with the same block, copied without adapting: only the listed files, no fixing what is wrong next door, no test, documentation, formatting or refactoring that was not requested, and a failing verification is a stop — never an adjustment of the verification. This is not decoration: without an explicit scope statement in the prompt, the rate of out-of-scope action in coding agents rises from close to zero to close to one in six.

**Proof by inventory.** Before starting, the execution records path, size and date of every file in `resources/`, and keeps that list in its own file. At the end of **each block** — not only at the end of the run — it compares. Everything that changed has to be in the union of the files declared in the executed tasks. The subagent's report is what it says it did; the inventory is what happened.

**The valve.** The executor will see wrong things that are not its own. It does not fix them: it records them in **Achados** (findings), with where it is and why it matters, marked as not executed. That is where the urge to fix becomes input for the next run instead of damage in this one. The `revisor` audits exactly that — a finding describing something in the same file the run changed outside the task is a disguised fix, and it is blocking.

## The reviewer

A subagent (`revisor`) with `tools: Read, Grep, Glob` that reads the artifact and the template, looks for defects in a **closed list** and reports by severity. It never fixes, never judges the merit of the decision, never comments on style, and it may — it should — answer `Nenhum defeito encontrado.` (no defect found).

The closed list is the point. A loose reviewer, with the mission of "seeing whether it can be improved", always finds something, because improvement is unlimited; the feedback becomes generic noise and around the third use you learn to ignore it. Each stage has its own list, with each defect named and the severity marked item by item: PDR with 8 items, SPEC with 14, PLAN with 15, TASKS with 22, EXEC with 20, ARCHIVE with 9. A stage that is not on the list it refuses instead of improvising a criterion.

And it works because it is blind: whoever wrote the artifact had the whole conversation in context, and for that reason cannot see what is missing. The reviewer does not receive the conversation, only the files — which is the condition of whoever will read it a year from now.

**The review is a loop, and it only ends clean.** The skill fixes the blocking and serious findings and calls the reviewer again, in a fresh invocation, until it returns zero of both. There is no maximum number of rounds, and before that nothing is delivered: the skill does not ask for your review and does not promote the `Status`, because there is no point in presenting to the author an artifact that is known to be defective.

What the skill cannot fix on its own reaches you as a question or as a block, never as an artifact with a defect inside. There are up to three cases: a finding that depends on an answer from you; a finding that came back after being fixed; and a finding that originates in the source artifact — that one blocks the stage until the source is fixed, because the skill does not edit an artifact that has already been approved.

In the execution, the reviewer audits the log, not the run: a log that records a failure faithfully is correct; the wrong one is the log that hides it. A fact the log cannot undo — contamination, violated order, an installation done by the executor — goes to **Desvios** (deviations) with your decision written on the line, and the run only closes as `concluída` (completed) with no pending fact: none without your decision, and none refused and not yet resolved.

Severity belongs to the item, not to the reviewer: each item on the list carries its own, so that zero blocking and zero serious is the same criterion in every round. An observation stays outside the loop: it reaches you, and you decide.

**Deviation from the original proposal.** Each audit checks the artifact against its immediate predecessor, and it is from stage to stage that the feature turns into something else without anyone having decided so. That is why, in the plan, in the tasks and in the execution, the reviewer also reads the PDR and the spec, and reports as blocking whatever contradicts the Decision, the Scope or the Out of scope of the PDR, or a premise in the "Viabilidade verificada" (verified feasibility) table — unless the artifact itself records the deviation.

## The gates

Every artifact carries `Status` at the top, and it is that field, not the memory of the conversation, that the next stage reads. The PDR is born `proposto` (proposed) and the spec requires `aceito` (accepted); the spec is born `rascunho` (draft) and the plan requires `aprovada` (approved); and so on.

The one who promotes is the skill itself, **after** the user approves in words. Never on its own, never treating silence as approval. An artifact that arrives with its status already advanced and no record of the approval is a blocking finding in every stage.

Approval is not a lock: an approved artifact is fixed by reopening it. In the first four stages, invoking the skill when its artifact already exists is a reopening — the `Status` goes back to the initial one, the fix goes through the reviewer's loop and through your approval again, and the later artifacts of the feature go back to `rascunho`, because they were written on top of what changed. Once execution has started, only the plan and the tasks reopen, and what has already been executed is not rewritten: the fix comes in as a new step or a new task. Changing the decision or the spec with code already written is a new feature. A change you ask for during review, instead of approving, also reopens the loop.

## What carries across features

`.spec/shared/memory.md` is the project's memory and the only file that is not cloned: it is filled in place, by `archive`, entry by entry, and only with your yes. Vocabulary, stack and pinned versions, conventions, invariants, limits and policies, permanent decisions, **paths already tried** and known pitfalls.

The PDR, the spec and the plan read it before any question, and what is there is not rediscussed at every round — contradicting it is a finding to report, not a choice to make. A section with fields between `<>` is empty: a placeholder is not a convention, not an invariant and not a decision.

At archiving, `archive` rereads what the feature taught and proposes what goes up. There is a single criterion: **does it hold for the next feature?** If it holds only for this one, it stays in the README of the archived folder. You approve entry by entry, and `archive` writes. Filling the memory with dead detail is the fastest way to make everyone stop reading it.

## Archiving

`archive` does not touch `resources/`. It checks the closing gate — log `concluída`, artifacts approved, every box checked, no pending fact —, **moves** the feature's artifacts from the root of `.spec/specs/` to `.spec/specs/archive/<NNN>-<slug>/`, writes there the README with the feature's memory, and proposes what goes up from it to the project's memory. It deletes nothing.

An abandoned feature is also archived, with fewer artifacts than the five, as long as the README of the folder records why it was abandoned and which stages never existed. Abandonment with a trail is legitimate; silent abandonment is not.

## Recovery

The execution refuses to start if there is already a run in progress — two orchestrators on the same feature corrupt each other's work. But a log in `Status: em execução` (running) with no live run is a **dead** run, not a run in progress: the skill shows the log to the user, asks for confirmation that nothing is running, closes that line as `interrompida` (interrupted) and resumes, skipping the tasks already checked. The trail of an earlier run is never erased — resuming adds a line to the Rodadas (runs) table.

That is why the `executor` checks state before acting: before creating, whether it already exists; before changing, whether it is not already as it should be. A task may be re-executed after an interruption, and running twice cannot duplicate anything.

## The rule almost everyone breaks

During execution you will discover that a step was wrong. When that happens, **stop and go up**: fix it in the source artifact and redo what depends on it. Fixing it directly in the code "just this once" is the exact point where SDD becomes dead documentation — and then you have paid the overhead without getting the benefit.
