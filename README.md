# Privasys agent skills

Reference skills for the [Privasys Harness](https://github.com/Privasys/harness).

A skill is **instructions, not privilege**. It cannot widen a tool set, reach
outside a grant, or spend past a policy cap, because all three are enforced in
the measured layer beneath the agent. That is what makes it safe to hand people
an editable behaviour file and call it theirs.

The deployment ships these read-only. They are copied once into the user's own
Drive, under `skills/` inside the folder that harness was granted (`AppData/
Harness/`, `AppData/Privasystant/`, whatever the deployment is called), and
from then on they are theirs: read before they run, edited in a text editor,
carried out through Drive export. The harness reads them back every pass, so
an edit in Drive is the behaviour of the next session. No build, no deploy.

| Skill | What it does |
|---|---|
| `inbox-triage/` | Reads what arrived, labels it in the user's own mail client, and leaves a draft where a reply is owed. Never sends. |
| `learn-writing-style/` | Reads the user's own sent mail and writes the style document every later draft is written against. |

Licence: MIT, so anyone can copy, edit and publish their own.
