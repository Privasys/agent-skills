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
| `inbox-triage/` | Triages what arrived since the last run (never "everything unread"), labels it in the user's own mail client, and drafts only where a reply is owed. Never sends. |
| `learn-writing-style/` | Reads the user's own sent mail and writes the style document every later draft is written against. |
| `agent-builder/` | Turns "make me an agent that…" into a folder under `agents/` in the user's own Drive, which the harness runs as a workspace and, when it has a trigger, unattended. Asks each consent in the conversation. |

An **agent** is one such folder: `agents/<name>/agent.md` (who it is),
`agent.yaml` (what starts it, what it needs, how often it may run) and an
optional `skills/` of its own. The harness mirrors it to a workspace beside
the user's others, starts runs on the trigger, and every run is a session
there. The chat builds and changes agents; a scheduled run never edits its
own definition.

Licence: MIT, so anyone can copy, edit and publish their own.
