---
description: Shared task board in the project's second brain. Plan and prepare tasks with dependencies and a ready-to-use brief, dispatch subagents for the free tasks, take one task in this session, show the board, or merge finished ones
argument-hint: "[ | N | tomar [T-NNN] | agregar <descripción> | preparar [T-NNN] | estado | mergear]"
---
Shared task board so several Claude sessions can work in parallel without stepping on each other and without re-reading the code. Reply to the user in their language. Note frontmatter keys, states and section headings use the Spanish schema below; write note prose in the language the vault already uses.

**Three roles:**
- **The planner** (`agregar`, `preparar`) reads the code once. It writes each task's dependencies, its track, and a **Base** good enough that a fresh agent can start working without exploring the repo.
- **The dispatcher** (no argument, or a number) runs in the owner's session. It claims free tasks, hands each one to a background subagent, reviews what comes back, and keeps the board. One session does the work of many tabs.
- **The taker** (`tomar`, or a dispatched subagent) starts from the Base and reads only what it lists.

Reading the code twice is waste. Avoid it.

## Where things live

- **Vault.** Resolve it like /brain-save: the entry in `${BRAINS_CONF:-$HOME/.claude/brains.conf}` (lines `<project folder name>|<absolute vault path>`, a leading `~` allowed, `#` comments and CRLF tolerated) whose project folder name equals a segment of the current working directory (the deepest such segment wins). `<vault>` below is the vault folder's name. If nothing matches, tell the user to run `/brain-init`.
- **Tasks.** `<vault>/Tareas/T-NNN <título corto>.md`, one note per task. The notes are the source of truth.
  - No `:` in names, because Obsidian forbids it. Use a comma.
- **Board.** `<vault>/Tareas/Tablero.md` is a derived view. Rebuild it from the notes after every change, and never edit it by hand.
- **Groups.** When `<vault>/Tareas/Grupos.md` exists, every task belongs to one group of its table, and each group has a priority (1 is the most important, `cerrado` when all its tasks are done). Its «Regla» section is binding:
  - a new task joins the group it fits; if none fits, add a group to the table with a priority and tell the owner where it went;
  - a new task in a `cerrado` group gives that group a priority again;
  - only the owner reorders existing priorities.
  - "By priority" below means: group priority first, then id.
- **Template.** `<vault>/_plantillas/Tarea.md`.
- **Plans and specs** that a task produces go where the project keeps them. Check the project's CLAUDE.md for where it keeps specs and plans; default to `docs/specs/` and `docs/plans/`. A taker writes them inside its worktree and commits them on the task branch, so the main checkout stays clean. Link them from the task.
- **Locks.** They live outside the synced repo: `~/.claude/state/tareas/<vault>/`. `mkdir` is the atomic claim. Locks are per machine, so never dispatch tasks for the same vault on two machines at once.
  - `$L` below means that folder. Shell variables don't survive between Bash calls, so start every call that uses it with `L="$HOME/.claude/state/tareas/<vault>"; mkdir -p "$L"`.
  - To release a lock, use guarded expansions: `rm -f "${L:?}/${ID:?}.lock/owner"; rmdir "${L:?}/${ID:?}.lock"`. The safety check refuses an unguarded `rm "$L/…"`.

## Task note format

### Frontmatter

| Field | Values and meaning |
|---|---|
| `id` | The task id. |
| `grupo` | Only in vaults with `Tareas/Grupos.md`: the exact name of the task's group there. |
| `estado` | `pendiente`, `preparando`, `en-curso`, `esperando`, `lista` or `hecha`. |
| `depende_de` | List of task ids. |
| `repos` | For example `[back]` or `[back, front]`. |
| `camino` | The track: `directo`, `plan`, `diseño` or `prototipo` (see «Tracks» below). |
| `tamaño` | `S` is under 1 h of agent work, `M` is a few hours, `L` should be split. |
| `preparada` | `true` once the Base is written. |
| `sesion`, `rama`, `tomada`, `actualizada` | Bookkeeping. Dates are absolute ISO date-times. |

### Body

- `## Objetivo`
- `## Contexto`: links to the vault's decisions and to records.
- `## Base`: the planner's brief. See below.
- `## Terminada cuando`: verifiable conditions.
- `## Registro`: append-only, one dated line per milestone.

### The Base: what the taker reads instead of the repo

- **Archivos.** Exact paths with line ranges and symbol names, because lines move before the task runs, and one line on what each holds. Name only the files the task touches or must read. State the commit the lines come from.
- **Enfoque.** The approach in a few lines: what changes where, which existing helpers or patterns to reuse (with path:line), and decisions already made. Cite the spec section it follows. The spec wins over the Base: when a Base must go past the spec, for example on permissions or money, say so and put it under «Para el dueño».
- **Tests.** Test files and test names, and the commands that prove it: focused tests, the full suite, typecheck and build.
- **Cuidado.** Traps, rulings and invariants from the vault (e.g. a `[[Trampas]]` note in `Recursos/`), plus anything another task touches in the same files.
- **Para el dueño.** Copy, decisions or approvals the owner must give, if any.

**Taker rule.** Trust the Base. Read only what it lists. Explore further only when the Base turns out to be wrong or incomplete. When that happens, fix the Base in the note, so the next agent benefits, and log it.

### States

- **`preparando`:** the planner is writing the Base. Takers skip it.
- **`esperando`:** the task needs the owner (a decision, a prototype or spec approval, credentials). The question goes in `## Registro`.
- **`lista`:** done on its branch, tests green, not merged.
- **`hecha`:** merged into `main`.

## Tracks (`camino`) — the planner decides

- **`directo`:** small and clear. The Base is enough to implement.
- **`plan`:** several files or steps, but no product decision left. The taker first writes a plan with superpowers:writing-plans, then executes it in the same task. If the plan has several independent parts, the planner should split them into separate tasks up front instead.
- **`diseño`:** there are product or architecture decisions only the owner can make. Make it a task of its own, «Diseño de X». Its deliverable is a spec, approved by the owner, written with superpowers:brainstorming. It ends `esperando` until the owner approves. The implementation tasks `depende_de` it.
- **`prototipo`:** the task changes UI the owner hasn't approved, so it needs a real-style prototype first (unless the project says otherwise). Either make a separate prototype task that the implementation depends on, or have the taker build the prototype first and set `esperando` until approval.

The planner splits large work into tasks: spec, then plan, then parallel implementation tasks with their dependencies.
- Prefer tasks that don't touch the same files, so they can run in parallel.
- When two tasks must touch the same file, make one depend on the other, or note it under «Cuidado».

## Mode: `agregar <descripción>` — plan and add tasks

1. **Understand the request.** Read the vault's hub, the relevant decisions and gotcha notes in `Recursos/`. Ask the owner only when scope or a dependency is genuinely unclear, one question at a time.
2. **Read the code once.**
   - For anything beyond a trivial lookup, delegate exploration to one Explore subagent. Give it a precise question per task: files, functions, line numbers, existing patterns, tests.
   - Keep its conclusions, not file dumps.
3. **Decide the tasks.** Choose the split, each task's `camino`, `tamaño`, `depende_de` and, with groups, its `grupo`. Pick dependencies on finished deliverables: a spec task, a migration, a shared helper.
4. **Pick ids.** Find the highest existing id, then claim `mkdir "$L/ids/T-NNN"`. If that fails, increment and retry.
5. **Write each note.** Use the template, with a complete Base and `preparada: true`. If a task must wait for an owner decision, set it `esperando` and write the question.
6. **Rebuild the board.** Then show the owner a short summary of:
   - the tasks;
   - the dependency order;
   - which ones can run in parallel right away;
   - what needs them.

**Owner-only work** (a Meta template, a payment, credentials, a deploy) is not a Claude task. Put it in the hub's `## Pendientes`.

## Mode: `preparar [T-NNN]` — write the Base for tasks that lack it

1. **Pick the targets.** Without an id, take every `pendiente` task with `preparada` missing or `false`.
2. **Lock each target.** Claim `mkdir "$L/T-NNN.lock"`. If that fails, skip the task. Then set `estado: preparando`.
3. **Explore.** Delegate the code reading, one Explore subagent per task, in parallel.
4. **Write the result.** For each task:
   - write the Base;
   - set `camino`, `tamaño` and `depende_de`;
   - split the task if it is `L`.
5. **Release.** Set `preparada: true` and `estado: pendiente`, or `esperando` if it needs the owner. Release the lock and rebuild the board.

## Mode: no argument, or `N` — dispatch subagents for the free tasks

**Cap.** At most **5 busy subagents at a time** by default, so the machine doesn't saturate (each one installs, tests and builds). Implementers and reviewers both count. An agent that reported `ESPERANDO` frees its slot until the owner answers. A number argument changes the cap for this run, e.g. `/tareas 3`. Never more than 5 unless the owner says so explicitly.

**Run id.** At the start, compute once `echo "$(date +%Y%m%dT%H%M%S)-$RANDOM-despacho"` and keep that literal value as RUN for the whole run (shell variables don't survive between calls). Every lock this run claims has `sesion=RUN`, and every note it takes gets `sesion: RUN`.

**Board.** Rebuild by hand per «Board format».

1. **Wait for merges.** If `$L/mergear.lock` exists, a merge is running and `main` is moving. Tell the owner and stop; run again when it ends.
2. **Find the free tasks.** Read every note in `Tareas/`, frontmatter only. A task is free with the rule in `tomar` step 2. Report separately:
   - stale tasks (rule at the end of `tomar`);
   - free tasks with `tamaño: L` that are not `diseño`. Don't run them; they need `preparar` to split them;
   - free tasks with `preparada` not `true`. Don't run them; offer `preparar`.
3. **Split the free tasks.** Read the Base of each free task, then sort them by id into two lists:
   - **For subagents:** `camino` `directo` or `plan`, and the Base asks nothing the owner must give before the commit: an approval, credentials, a push, prod access or a decision. Steps after the merge don't count, such as deploying, running a migration in prod, or purging a cache, because those are always the owner's.
   - **For the owner's session:** everything else, i.e. `diseño`, `prototipo`, and tasks that need the owner before the commit. Subagents can't talk to the owner.
4. **Pick the wave.** Go through the subagent list by priority (id order when the vault has no groups) until the cap is full. A task conflicts with another task, running or already picked, when they share a repo and either:
   - one Base's «Cuidado» names the other's id (with or without a link);
   - or both tasks modify the same file.

   To compare files:
   - Use full repo-relative paths. Resolve a bare file name to its path when it is unique in the repo.
   - Count only files the task modifies. Ignore files listed only to read or copy a pattern from («patrón», «no cambiar», «referencia»).
   - Ignore a copy catalog where each task adds its own block, such as an i18n file. Tell each taker to keep its copy in its own block; `mergear` resolves any textual conflict. Counting it would run almost every UI task one at a time.
   - When a Base says to find the files by search, run that search (`rg -l`) and use the result.

   Running tasks are those `en-curso` or `esperando`, in any session. `lista` tasks don't count: `mergear` resolves their conflicts. Skip conflicting tasks; they stay in the queue.

   Migrations don't make tasks conflict. The dispatcher reserves a distinct number for each task that adds one.
   - The next free number is the highest of: `main`, every `tarea/*` branch, every reservation in `$L/migraciones/<repo>/`, and any number written in the Base of a task that is not `hecha`.
   - Reserve it with `mkdir "$L/migraciones/<repo>/NNNN"` and an `owner` file holding the task id. If the `mkdir` fails, try the next number.
   - Put the number in the task's prompt.
5. **Claim, in this session, one task at a time:**
   - lock as in `tomar` step 3, with `sesion=$RUN`;
   - mark it taken as in `tomar` step 4, without rebuilding the board yet;
   - create its worktrees as in `tomar` step 6. Do it here and sequentially, because concurrent `git worktree add` on the same repo can collide.

   Then rebuild the board once.
6. **Dispatch.** Launch one `general-purpose` subagent per task, in the background, all in one message, with the prompt in «Subagent prompt». Then tell the owner, in a few lines:
   - what is running;
   - what is queued;
   - what is for the owner's session (`diseño`, `prototipo`, needs the owner);
   - what is stale or `L`.

   Offer in one line to take the first owner task here with `tomar` while the subagents work.
7. **When a subagent reports back,** act on its first word. A notice that an agent is waiting on its own background work is not a report, and the same report can arrive twice: act once.
   - **`LISTA`:** dispatch one fresh reviewer subagent for the whole task, following superpowers:requesting-code-review. Give it the diff of each worktree against `main`, the note's «Objetivo», «Base» and «Terminada cuando», and the spec the Base cites. Ask it to check permissions and money against the spec, not the Base, and to say which findings should block publishing.
     - If the reviewer finds real problems, send them to the same implementer with SendMessage and wait for a new report. Decide each finding yourself: fix it, accept it with a line in the note, or open a follow-up task with `agregar`. Ask the owner only about product decisions.
     - Treat an automated security review of a task's commits as a reviewer finding: verify it, then send it to the implementer.
     - When the fixes touch permissions, money, saving or a public route, send only the fix commits to the same reviewer, and repeat until no important finding is left. Other fixes need no new review.
     - When the review is clean:
       - set `estado: lista` and `actualizada`;
       - log the commits, the test counts and the review result;
       - release the lock;
       - rebuild the board.
   - **`ESPERANDO`:** the note is already `esperando` with the question. Rebuild the board. Ask the owner, one question at a time, and keep the lock. When the owner answers:
     - set the note back to `en-curso`;
     - log the answer;
     - send it to the same agent with SendMessage.
   - **`BLOQUEADA`:**
     - log the reason;
     - remove the worktrees with `git worktree remove` on their literal paths;
     - keep the branch and `rama`, so the next taker reuses them;
     - set `estado: pendiente`;
     - release the lock;
     - tell the owner.

   After each report, if a slot is free, go back to step 2: re-scan the whole board, not just the old queue, because merges and finished designs free new tasks.
8. **Owner's session in parallel.** If the owner is doing a `tomar` task here, the conversation with them comes first. Handle subagent reports between turns, and mention them in one line.
9. **Finish.** When nothing is queued or running, rebuild the board. Summarize in a few lines:
   - what is `lista`;
   - what is waiting for the owner;
   - what was blocked.

   Offer `mergear`. Tasks that depend on a `lista` task only free up after the merge.

Never merge, push or deploy in this mode, unless the owner delegated the run (below).

### When the owner delegates the whole run

The owner may ask to finish everything and publish while away. That authorizes this run only, and exactly what they named: merging, pushing, publishing, shutting down.

- **Say your rules once, right after they ask:** what you will not take, and the publishing cutoff.
- **Keep out tasks whose publishing could hurt real users with nobody watching:** billing, limits or pausing accounts, real payments, removing a safety guard, paid messages. A task's own «Para el dueño» can also keep it out.
- **Merge between waves** with `mergear` when dependencies need it. Run the full suites at the end of a wave, before dispatching tasks from that `main`.
- **Set a cutoff from the project's deploy window,** and start a background timer for it: if an agent hangs, no notice wakes you. Tell long tasks their deadline, and what to finish first if time runs short. What isn't merged by the cutoff stays out.
- **Publish by the project's deploy steps,** in their order. Do not retry or work around a denied permission: record it and continue.
- **Log every decision you made for the owner** in the vault, to confirm, and everything not tried on a real device.
- **Save to the vault before shutting down.** Schedule a shutdown last, with a few minutes of margin and how to cancel it.

### Subagent prompt

Fill in the brackets. Send exactly these parts, in this order:

```
You are the taker of task [T-NNN] on the /tareas board. The task is already claimed for you by a dispatcher.

Note: [absolute path of the note]
Worktrees (work only here, never in the main checkout): [repo → absolute path, branch]
Migration numbers reserved for you: [numbers, or "none"]
Merged since the Base was written: [tasks, and what moved or changed for this one, or "nothing"]
Running at the same time: [tasks and the areas they touch, or "nothing"]
Environment: [how to run the package manager, tests and browser checks on this machine; known failures that are not this task's]
Scratch files: only under a folder named after your task in the scratchpad. It is shared between agents, so never reuse a generic path like `api.log` or `export`.
Background: the plugin's `/tareas` command (commands/tareas.md), sections "The Base", "Taker rule" and "Tracks". The steps below replace "Mode: tomar" for you.

1. Read the note. Trust its Base. Read only what it lists. Its line numbers may be old: search by symbol. If the Base is wrong or incomplete, fix it in the note and log it.
2. If `preparada` is not true, write the Base yourself first, reading the code directly. Do not lock anything and do not change `estado`.
3. Follow the `camino`:
   - directo: tests first;
   - plan: write the plan with superpowers:writing-plans, save it inside your worktree in [plans folder, e.g. docs/plans/], commit it on the task branch, link it in the note, then execute it yourself with superpowers:executing-plans.
   Do not launch subagents.
4. On each milestone, update `actualizada` and append a line to `## Registro` of your note, with the real time from `date`, never an estimate. Edit only your own note. Do not touch the lock or the board.
5. When «Terminada cuando» holds and focused tests, full suite, typecheck and build pass, commit on the task branch with conventional commits. Do not merge, push or deploy. Do not run a code review: the dispatcher does it, and may send you fixes. Keep new copy in its own block of the copy catalog.
6. If you need the owner (a product decision, credentials, an approval), set `estado: esperando`, write the question in `## Registro`, and stop.

Your final message starts with exactly one of these words, then the details:
- LISTA, then: branch per repo, commits, test/typecheck/build summary with counts and the commit each ran on, what you did not check, any Base corrections, and any copy of yours the owner has not seen.
- ESPERANDO, then: the single question for the owner and what you already tried.
- BLOQUEADA, then: what blocks it and what you tried.
```

## Mode: `tomar [T-NNN]` — take one task in this session

Use it for `diseño` and `prototipo` tasks, or when the owner wants one task done here. With an id, take that task if it is free; otherwise take the first free one.

1. **Read the notes.** Read every note in `Tareas/`, frontmatter only.
2. **Pick a candidate.** It is the first task, by priority (by id when the vault has no groups), where:
   - `estado` is `pendiente`;
   - every `depende_de` task is `hecha`. A dependency on a `diseño` task is satisfied when that task is `hecha`, which means its spec is approved.

   Skip tasks with any dependency that is not `hecha` (or does not exist), and say which ones you skipped and why. Don't take a task with `tamaño: L` unless it is `diseño` (its spec splits the work); offer to split it with `preparar`. If no task is free, show the board summary and stop.
3. **Claim it atomically:**
   ```bash
   L="$HOME/.claude/state/tareas/<vault>"; mkdir -p "$L"
   mkdir "$L/T-NNN.lock" && printf '%s\n' "sesion=$(date +%Y%m%dT%H%M%S)-$RANDOM" "cwd=$PWD" > "$L/T-NNN.lock/owner"
   ```
   If `mkdir` fails, another session owns the task. Go back to step 2 with the next candidate.
4. **Mark it taken.** In the note, set:
   - `estado: en-curso`;
   - `sesion`, `tomada`, `actualizada`;
   - `rama`.

   Append a line to `## Registro`, then rebuild the board.
5. **Prepare it if needed.** If `preparada` is not `true`, write the Base first, the same way as `preparar`, then continue.
6. **Work in isolation.** For each repo in `repos`, create a worktree on branch `tarea/T-NNN-<slug>` from `main`, under `<project>/.worktrees/<repo>-tNNN`. The slug is a few words of the title, Spanish, lowercase, hyphens, no accents. If the note already has a `rama` whose branch exists, reuse it (`git worktree add <path> <branch>`, without `-b`). Never work in the main checkout.
7. **Follow the `camino`:**
   - **`directo`:** implement from the Base. Tests come first.
   - **`plan`:** write the plan with superpowers:writing-plans, commit it on the task branch and link it in the note. Then execute it in this session. For big plans, use superpowers:subagent-driven-development.
   - **`diseño`:** brainstorm with the owner (superpowers:brainstorming) and write the spec. Set the task `esperando` until the owner approves; once approved, it is `hecha`. Its dependent tasks then become free. Before finishing, fill those tasks' Bases from the spec, so their takers start ready.
   - **`prototipo`:** build the prototype, try it in a browser against what its notes claim, then publish it and set the task `esperando` until the owner approves. If it offers alternatives and the approval doesn't name one, say which you take before implementing. A fix after approval is reported, not assumed approved. Write the approved choices into the Base, then implement.
8. **Keep to the project's rules:**
   - a fresh reviewer before calling the work done;
   - never deploy;
   - never write secrets to the vault.
9. **Heartbeat.** On each milestone, update `actualizada` and append a line to `## Registro`.
10. **When the owner is needed,** set `estado: esperando`, write the question, ask the owner, and keep the lock.
11. **Finish.** When «Terminada cuando» holds and the tests pass:
    - commit on the task branch;
    - set `estado: lista`;
    - log the commits and the test summary;
    - remove the lock: delete its `owner` file, then `rmdir` the lock;
    - rebuild the board.

    Do NOT merge. Report the result in a few lines and offer to take the next task.

**Stale tasks.** A task that has been `en-curso` or `preparando` with `actualizada` older than 6 hours may be abandoned. Show it to the owner and ask before taking it over.

## Mode: `estado` — show the board

Rebuild `Tablero.md`, then show:
- counts per state;
- what is free to take now;
- what is blocked, and on what;
- what is waiting for the owner;
- what is ready to merge;
- which pending tasks are not prepared yet.

## Mode: `mergear` — merge finished tasks

Run this only when the owner asks.

1. **Claim the merge lock** with `mkdir "$L/mergear.lock"`.
2. **Merge each `lista` task, in dependency order:**
   - update `main`;
   - merge the task branch with `--no-ff` and a conventional message;
   - resolve conflicts;
   - run typecheck plus only the test files related to what the merge touched (the branch's own tests and those of conflicted files);
   - if they fail, stop that task: leave it `lista`, log why, and continue with the others that don't depend on it.

   After the last merge, run the repo's full tests and build once. If they fail, find the merge that broke it (bisect over the merge commits), revert that merge, set its task back to `lista` with the reason, and rerun.

   A branch that already contains `main` needs neither run, if its full suites passed on its final code: confirm `git diff <branch> HEAD` is empty after the merge, and say you relied on that.

   Don't run `mergear` while implementers are testing on the same machine: suites slow each other down several times over. Wait for them, or pause dispatching.
3. **For each merged task:**
   - set `estado: hecha` and log the merge commit;
   - release its migration reservations: delete the `owner` file, then `rmdir` each `$L/migraciones/<repo>/NNNN` it holds;
   - remove its worktree with `git worktree remove` on the literal path. On Windows it can fail with «Filename too long» on `node_modules`, leaving the folder: delete that literal path with Node's `fs.rmSync(path, { recursive: true, force: true })`, from a shell that isn't inside it, then `git worktree prune`;
   - delete the branch with `-d`.
4. **Finish.** Remove the merge lock and rebuild the board. Report what merged, what didn't and why, and which tasks are now unblocked.

Never push or deploy unless the owner asks.

## Board format (`Tablero.md`)

- Frontmatter: `hub: "[[Inicio]]"`.
- At the top, one line: «Generado por /tareas el <fecha>. No editar a mano.»
- One table per state, in this order: `en-curso`, `preparando`, `pendiente`, `esperando`, `lista`, then `hecha` (last 15 only).
- Columns: `Tarea` (link `[[T-NNN título]]`), `Camino`, `Tamaño`, `Depende de`, `Repos`, `Sesión`, `Actualizada`.
- In the `pendiente` table, mark unprepared tasks with «(sin preparar)».
- With groups, the board has instead one section per active group in priority order, each with its unfinished tasks (in progress first, then free, then blocked), a «Grupos cerrados» list, a «Sin grupo» table if any task lacks a valid `grupo`, and the last 15 `hecha`.
