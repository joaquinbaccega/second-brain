---
description: Create a second-brain vault (Obsidian) for the current project and register it in brains.conf
argument-hint: "[vault path — optional]"
---
Create a second-brain vault for the current project. Never install software; only create files.

1. **Vault path.** Use `$ARGUMENTS` if given. Otherwise ask the user for an absolute path (suggest a folder outside the repo, e.g. `~/brains/<project folder name>`) and wait for the answer. Expand a leading `~`. If the path already contains `Inicio.md`, do not overwrite anything: skip to step 4.
2. **Create the structure** at that path from the plugin templates in `${CLAUDE_PLUGIN_ROOT}/templates`:
   - folders `Decisiones/`, `Recursos/`, `Tareas/`, `_plantillas/`;
   - `Inicio.md` from `templates/Inicio.md` (replace "Project name" with the project folder name);
   - `Tareas/Tablero.md` from `templates/Tablero.md`;
   - `_plantillas/Decision.md` and `_plantillas/Tarea.md` from the templates of the same name;
   - `Modelos.md` from `templates/Modelos.md`.
   Never overwrite an existing file.
   Then ask the user which models `/tareas` subagents should use, offering the perfiles of the table in `Modelos.md` with one line each, `equilibrado` first as the recommended default. Set `perfil` to the answer; for `propio`, write the user's rules under «Reglas propias». This can be changed later with `/tareas modelos`.
3. **Fill the hub lightly.** If the project has a README or CLAUDE.md, fill `## Qué es` and `## Stack` of `Inicio.md` with one or two lines each. Do not invent anything else.
4. **Register it.** The conf file is `${BRAINS_CONF:-$HOME/.claude/brains.conf}`. Create it (and its folder) if missing. Append the line `<current project folder name>|<absolute vault path>`, where the folder name is the basename of the project root. Do not add the line if an identical one already exists; if the same project name maps to another path, show both and ask which to keep.
5. **Tell the user** to open the vault in Obsidian: "Open folder as vault" and pick the vault path. The hub is injected automatically at the next session start in this project; `/brain-load` loads it now.
