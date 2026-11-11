---
description: Load the full second brain (Obsidian vault) of the current project
argument-hint: "[vault name — optional]"
---
Resolve the vault: `$ARGUMENTS` if given; otherwise the entry in `${BRAINS_CONF:-$HOME/.claude/brains.conf}` (lines `<project folder name>|<absolute vault path>`, a leading `~` allowed, `#` comments and CRLF tolerated) whose project folder name equals a segment of the current working directory (the deepest such segment wins). If none matches, say so, list the vaults configured in the conf file, and suggest `/brain-init`.

1. Read `Inicio.md` (the hub) and every note linked under its `## Decisiones`.
2. List `Recursos/`; read the notes relevant to the current task.
3. Reply with a 5-line max summary: current state, active decisions that constrain work, open pending items. Do not dump note contents.
