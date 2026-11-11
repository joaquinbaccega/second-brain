---
description: Save a decision, learning or gotcha from this session to the project's second brain
argument-hint: "[what to save — optional]"
---
Resolve the vault: the entry in `${BRAINS_CONF:-$HOME/.claude/brains.conf}` (lines `<project folder name>|<absolute vault path>`, a leading `~` allowed, `#` comments and CRLF tolerated) whose project folder name equals a segment of the current working directory (the deepest such segment wins). If nothing matches, tell the user to run `/brain-init`.

What to save: `$ARGUMENTS`. If empty, pick the durable items from this conversation: decisions made, root causes of bugs, non-obvious gotchas, preferences. Skip anything already in the code or git history, and anything only relevant to this session.

Vault layout: `Inicio.md` (hub: what it is, stack, production, decisions list, pending), `Decisiones/`, `Recursos/`, `Tareas/`, `_plantillas/`.

For each item:
1. Search the vault (`rg -il`) for a note that already covers it. Update it instead of duplicating.
2. Decision → `Decisiones/YYYY-MM-DD titulo-corto.md` from `_plantillas/Decision.md` (context, decision, why, discarded alternatives), and link it in the hub's decisions list as `- [[nota]]: what it constrains, in one line`. If it supersedes an older decision, set the old one's `estado: reemplazada`, add a `> Reemplazada por [[nueva]]` line under its title and `> Reemplaza a [[vieja]]` in the new one, and move the old one to the hub's «Reemplazadas» line.
3. Gotcha / how-to → a note in `Recursos/`, linked from the hub. If the note has sections (e.g. `Trampas`), add the item to its section.
4. State change → the hub is a snapshot, not a log. **Replace** the `## Producción` lines that changed (deployed commit, migrations, update ids) instead of adding a paragraph, and prepend the old state, dated, to `Recursos/Bitácora.md` (create it if missing). Remove `## Pendientes` items that are done or past their date (move them to the Bitácora's «Pendientes superados»). Work for Claude goes to the task board (`/tareas agregar`), not to the hub's pending list.
5. Size: the SessionStart hook injects only the first 8000 bytes of the hub. Keep `Inicio.md` under 7000 bytes (`wc -c`); if it grows past that, move history to the Bitácora before finishing.

Rules: links as `[[Note name]]`; absolute dates; write note prose in the language the vault already uses (default: the user's language), keeping the Spanish schema keys and headings; short and scannable. Never store secrets, tokens, ID documents or patient data — note where they live instead. Never delete notes.
Finish with the list of files created or updated.
