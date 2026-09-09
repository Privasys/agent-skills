# Privasys agent skills

Reference skills for the [Privasys Harness](https://github.com/Privasys/harness).

A skill is **instructions, not privilege**. It cannot widen a tool set, reach
outside a grant, or spend past a policy cap, because all three are enforced in
the measured layer beneath the agent. That is what makes it safe to hand people
an editable behaviour file and call it theirs.

The deployment ships these read-only. The first time a user runs one, it is
copied into their own Drive under `AppData/Harness/skills/`, and from then on it
is theirs: they can read it before it runs, edit it, and carry it out through
Drive export.

| Skill | What it does |
|---|---|
| `learn-writing-style/` | Reads the user's own sent mail and writes the style document every later draft is written against. |

Licence: MIT, so anyone can copy, edit and publish their own.
