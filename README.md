# AGENTS.md Protocol: Central Command

```
 █████╗  ██████╗███████╗███╗   ██╗████████╗ ██████╗
██╔══██╗██╔════╝██╔════╝████╗  ██║╚══██╔══╝██╔════╝
███████║██║  ███╗█████╗  ██╔██╗ ██║   ██║   ███████╗
██╔══██║██║   ██║██╔══╝  ██║╚██╗██║   ██║   ╚════██║
██║  ██║╚██████╔╝███████╗██║ ╚████║   ██║   ██████╔╝
╚═╝  ╚═╝ ╚═════╝╚══════╝╚═╝  ╚═══╝   ╚═╝╚═════╝
```

---

## ◆ PULSE

An agent without a contract is a generator of surprises. This
repository is the central command for the `suradet-ps` ecosystem: a
curated collection of immutable contracts, architectural constraints,
and phase-aware protocols that standardize how autonomous AI agents
build software across every project here. Each contract follows the
same anatomy - context and role, immutable constraints, phase-aware
workflow, negative constraints, quality gates - and every task file
carries its status in its name: `01-TODO-...` waits, `01-DONE-...`
is archived context.

| 9 projects ▣ | TODO/DONE ▣ | 5-part anatomy ▣ | Binding ▣ |
|---|---|---|---|

*The command center - contracts, sequences, gates - is sealed.*

> Maintained by **suradet-ps** - control the agent, control the code.
>
> **suradet-ps**, artifact keeper

---

## ◆ IGNITION

One file, one bootstrap prompt.

```
⟫ git clone https://github.com/suradet-ps/agents-md.git
⟫ cd agents-md
```

Open the project folder, take the target `.md` file, and prepend the
activation instruction:

> **SYSTEM PROMPT:** "You are an autonomous developer agent. Read the
> attached `AGENTS.md` file. This file is a **BINDING CONTRACT**. You
> must strictly follow the constraints, stack, and phase-aware
> workflow defined within. Do not deviate. Confirm your understanding
> of the **Primary Objective** before generating code."

That is the whole ritual: pick a contract, bind the agent, start the
work.

---

## ◆ ANATOMY

One convention, five mandatory parts, zero ambiguity.

- **Names** - every file is `[Sequence]-[Status]-[Slug].md`:
  sequence orders the execution, status says whether it waits
  (`TODO`) or serves as archived context (`DONE`) - agents process
  tasks in order and never execute what is done.
- **Contexts** - each contract opens with context and role: who the
  AI is in this task - a Senior Rust Architect, a Vue 3 Specialist -
  so the work begins with the right posture.
- **Binds** - immutable constraints follow: the facts that cannot
  change, from "Vue 3 Composition API" to "no jQuery" - written so
  deviation is visible on sight.
- **Sequences** - the phase-aware workflow runs in order: Analysis,
  Approval, Code, Verify - a task never skips its own gates.
- **Forbids** - negative constraints say what is explicitly banned:
  no `any` types, no direct DOM manipulation - the list of what
  "done" must not contain.
- **Gates** - quality gates close each contract: the tests and
  linters that must pass before a task is allowed to call itself
  done.

---

## ◆ RITUALS

**The core ceremony** - deploying a new task:

1. Find the highest sequence number in the project folder.
2. Create `[Next_Number]-TODO-[task-name].md` - atomic, self-contained,
   one task per file.
3. Prepend the bootstrap prompt when activating the agent; the
   contract binds before the code flows.
4. When the gates pass, rename to `DONE` - the task becomes archived
   context, a lesson for the next agent.

**The ceremony of the sequence** - numbers order the work; status
words guard the archive. An agent reading the folder knows what to
do next without asking, and knows what not to touch without being
told.

**The ceremony of the immutable** - the contract says what cannot
change, and the agent confirms understanding before generating a
single line. The constraint is not a suggestion wearing a title; it
is a binding clause.

---

## ◆ ECHOES

**Where this artifact is heading**

```
projects ▸ 9 folders: drug-cup-online to warfarin-app ──────────────── ▸ sealed
protocol ▸ 5-part anatomy, phase-aware workflows ───────────────────── ▸ sealed
naming   ▸ sequence-status-slug convention ─────────────────────────── ▸ sealed
```

**Raising the artifact** - a new project adds a kebab-case folder, a
`README.md` describing its stack and goals, and agent files starting
at `01-TODO-...`. A new task follows the sequence ritual. Open an
issue first to discuss a change to the protocol itself.

**Status** - the protocol is active and in use across the ecosystem.

---

```
  ─────────────────────────────────────────
   Control the Agent, Control the Code.
  ─────────────────────────────────────────
```

Maintained by suradet-ps.