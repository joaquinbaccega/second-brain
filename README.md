# second-brain

A per-project "second brain" for Claude Code. Each project gets an Obsidian vault. At session start the vault's hub note is injected into the context, so Claude knows the project's decisions and state. Commands save new knowledge and run a shared task board.

## Install

```
/plugin marketplace add joaquinbaccega/second-brain
/plugin install second-brain@second-brain
```

`second-brain` declares `superpowers` (from `claude-plugins-official`) in its `dependencies`, so Claude Code installs it automatically with the plugin. If it does not, run `/plugin install superpowers@claude-plugins-official`.

Nothing in this plugin installs software on your machine.

## Requirements

- `bash` (on Windows: Git Bash, from Git for Windows).
- The `superpowers` plugin, needed by `/tareas` (writing-plans, executing-plans, brainstorming, subagent-driven-development).
- Obsidian (optional, to browse and edit the vault; the vault is plain Markdown).

## Quick start

1. Open Claude Code in your project and run `/brain-init` (optionally with a vault path).
2. Open the vault folder in Obsidian ("Open folder as vault").
3. Start a new session in the project: the hub (`Inicio.md`) is injected automatically.

## `brains.conf`

Maps projects to vaults. Location: `${BRAINS_CONF:-$HOME/.claude/brains.conf}`. One line per vault:

```
# <project folder name>|<absolute path to vault>
my-app|~/brains/my-app
other-app|/home/me/notes/other-app
```

- A project matches when one segment of the working directory equals the project folder name; the deepest match wins.
- A leading `~` is allowed. `#` comments and CRLF line endings are tolerated.
- With no conf, no match or no `Inicio.md`, the hook does nothing.

## Commands

| Command | What it does |
|---|---|
| `/brain-init [path]` | Creates a vault from the templates and registers it in `brains.conf`. |
| `/brain-load [vault]` | Reads the hub and its decisions and summarizes the current state. |
| `/brain-save [what]` | Saves decisions, learnings and gotchas as notes and updates the hub. |
| `/tareas [...]` | Shared task board: `agregar`, `preparar`, dispatch (no argument or `N`), `tomar`, `estado`, `mergear`. |

## Note schema

The schema keeps Spanish keys on purpose; it is the convention all commands rely on.

```
<vault>/
  Inicio.md        hub: qué es, stack, producción, decisiones, pendientes (keep under 7000 bytes)
  Decisiones/      YYYY-MM-DD titulo.md   (estado: vigente | reemplazada)
  Recursos/        how-tos, gotchas, Bitácora.md
  Tareas/          T-NNN titulo.md + Tablero.md (derived view)
  _plantillas/     Decision.md, Tarea.md
```

Task notes use `estado` (`pendiente`, `preparando`, `en-curso`, `esperando`, `lista`, `hecha`), `depende_de`, `camino` (`directo`, `plan`, `diseño`, `prototipo`), `tamaño`, `repos`.

## Limits

- Task locks live in `~/.claude/state/tareas/<vault>/` and are per machine. Do not dispatch tasks for the same vault from two machines at once.
- The SessionStart hook injects only the first 8000 bytes of the hub.

## Setup instructions for an AI agent

Checklist for a Claude Code session to set this up end to end. Steps marked **[HUMAN]** need the person; ask them and wait. Do not skip the confirmations, and do not install anything the person has not approved.

1. **Detect the OS.** Run `uname -s` (Linux, Darwin, or MINGW/MSYS/CYGWIN for Windows Git Bash); on native Windows use PowerShell. Tell the person what you detected.
2. **Obsidian (optional, ask first) [HUMAN approval]:**
   - macOS: `brew install --cask obsidian`
   - Windows: `winget install Obsidian.Obsidian`
   - Linux: `flatpak install flathub md.obsidian.Obsidian`, or download the AppImage from https://obsidian.md/download
   Skip if the person already has it or does not want it.
3. **Ensure bash.** macOS and Linux already have it. On Windows check `bash --version`; if missing, with the person's approval run `winget install Git.Git` (provides Git Bash), then **[HUMAN]** restart the terminal.
4. **Add the marketplace and install** (inside Claude Code):
   ```
   /plugin marketplace add joaquinbaccega/second-brain
   /plugin install second-brain@second-brain
   ```
   CLI equivalents from a shell:
   ```
   claude plugin marketplace add joaquinbaccega/second-brain
   claude plugin install second-brain@second-brain
   ```
5. **Confirm superpowers.** It should arrive as a dependency. Check with `claude plugin list`; if absent, run `claude plugin install superpowers@claude-plugins-official`.
6. **[HUMAN] Reload.** Run `/reload-plugins`, or restart Claude Code, and approve any plugin trust prompts.
7. **Create the vault.** In the project directory run `/brain-init` and give it a vault path (ask the person where; suggest `~/brains/<project>`).
8. **[HUMAN] Open the vault in Obsidian.** Obsidian, "Open folder as vault", choose the vault path. This is a GUI step.
9. **Verify.** **[HUMAN]** Start a new Claude Code session in the project. Check that the context starts with `## Second brain: <vault>` followed by the contents of `Inicio.md`. If not: confirm the `brains.conf` line names the project folder (not a path) and the vault has `Inicio.md`; test the hook with `CLAUDE_PROJECT_DIR=$PWD bash <plugin dir>/hooks/brain-context`.

## License

MIT
