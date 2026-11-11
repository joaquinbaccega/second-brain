---
perfil: equilibrado
---
# Modelos

Which model each `/tareas` subagent uses (explorers, implementers, reviewers). Change `perfil` above, or run `/tareas modelos <perfil>`. A task done with `tomar` runs in your own session and uses that session's model.

## Perfiles

| perfil | Sonnet | Opus | Fable |
|---|---|---|---|
| `sonnet` | everything | never | never |
| `opus` | never | everything | never |
| `fable` | never | never | everything |
| `ahorro` | everything else | only to review sensitive tasks | never |
| `equilibrado` | the default: small or contained tasks, copy, screens, small refactors, simple Bases, reviews of small diffs, code exploration | sensitive tasks and their reviews | only when Opus is not enough |
| `calidad` | only code exploration | the default | sensitive tasks and their reviews |
| `propio` | see «Reglas propias» | | |

**Sensitive** means: money (payments, refunds, billing, subscriptions), permissions and security, migrations or concurrency (locks, transactions), many steps or several layers at once, and architecture design.

**Fable in `equilibrado`**: high-stakes architecture, end-to-end money flows, long multi-layer tasks where a mistake is expensive, or a task Opus already tried and did not solve well. Never small tasks or small reviews.

Each tier costs several times the one below it (Sonnet < Opus < Fable).

## Reglas propias

Used only when `perfil: propio`. Write the rules in your own words, for example: "Opus for anything that touches the API, Sonnet for the rest, Fable never".
