---
name: agent-builder
description: Turn "make me an agent that…" into a folder in the user's own Drive that the harness runs as a workspace, unattended when it has a trigger. Asks for each consent in the conversation. Never edits an agent from inside one of its scheduled runs.
---

# Agent builder

An agent is not code and not a deployment. It is a folder in the user's own
Drive, under the assistant's app folder:

```
AppData/<assistant folder>/agents/<Agent name>/
  agent.md        who the agent is and its standing instructions
  agent.yaml      what a run does, what starts it, how often it may run
  skills/         skills specific to this agent (optional)
  state/          written by runs, never by you
  runs/           written by runs, never by you
```

The harness mirrors that folder into a workspace of its own, beside the
user's other workspaces, within about fifteen seconds of a change. The
user's chat can open the workspace and talk to the agent; when `agent.yaml`
declares a trigger, the harness also starts runs unattended and each run is
a session in that workspace, where the user reads it like any other.

Use this skill when the user asks for an assistant that does something on
its own ("triage my inbox every morning", "every Friday, list what I
committed to this week", "when mail arrives from the board, draft an
acknowledgement"), or when they ask to change an agent they already have.

## 1. Agree on the agent in the conversation first

Ask, briefly, and only what the template below cannot default:

- **Name.** One plain phrase, it becomes the workspace title: "Inbox triage",
  "Weekly commitments". No slashes, no leading dot, sixty characters at most.
- **What a run does**, in one or two sentences. This becomes `prompt`.
- **What starts it.** One of: `every: 2h` (a Go duration: `30m`, `2h`,
  `24h`), or `on: mail.changes` (a message arrived in the linked mailbox).
  Both may be set. If they want a time of day, say that only intervals and
  arrivals are supported for now and pick the nearest interval.
- **What it needs.** The resources its runs will use, by kind, from what
  `list_access` reports: `mail.mailbox` for the mailbox, `storage` for
  Drive. This is not written in the folder; it is what you ask consent
  for in section 3. Do not ask for what the run will not use.
- **How careful.** Defaults are right for almost everyone: `debounce: 2m`
  (a burst of arrivals becomes one run), `min_interval: 10m` (never more
  often than this, whatever arrives, and one run at a time).

Read back the agreed definition in prose before writing anything. Keep it to
what was agreed; do not invent behaviour the user did not ask for.

## 2. Write the folder through Drive's own tools

The assistant's app folder is `AppData/<label>/`, where `<label>` is the
`label` of the `storage` row in `list_access` (for example `Privasystant`
or `Harness`). Navigate with `list_root` and `list_folder` to `AppData`,
then the label, then `agents`; create `agents` with `create_folder` if it
does not exist, then the agent's folder inside it. Files are written with
`write_file` (`parent_id`, `name`, `content_base64`, `mime: text/markdown`
or `text/yaml`). Encode the content as base64 yourself.

Write exactly these two files. Create `skills/` only if the agent needs a
skill of its own; do not create `state/` or `runs/` (the harness does).

`agent.md`:

```markdown
# <Agent name>

You are <the user's first name>'s <what the agent is for>, running in the
Privasys Harness on their behalf.

## What you do

<The agreed job, in the user's words, as instructions.>

## How you work

- Follow the `<skill name>` skill for the steps and the taxonomy.
- Write only through the tools you were granted; never send mail.
- Write nothing outside this workspace's `state/` and `runs/` folders.
- Never change `agent.md`, `agent.yaml` or `skills/` from a run. If the
  definition should change, say so in the run's summary and stop.
- Finish every run with a short summary: what arrived, what you did, what
  needs the user.
```

`agent.yaml`:

```yaml
prompt: <one or two sentences: what a run does>
trigger:
  on: mail.changes        # or: every: 2h   (both may be set)
debounce: 2m
min_interval: 10m
paused: false
```

If a Drive tool refuses because this workspace's Drive knowledge is off or
narrowed, say so: the agent has to be written from a workspace that may
reach Drive, and the user chooses that under the workspace's menu.

## 3. Ask for each consent, here, now, in the right order

For every resource kind agreed in section 1, look at `list_access`. If it
is not `approved`:

1. **First make sure there is something to approve.** For `mail.mailbox`,
   call the mail tool `account` before anything else. If it says no mailbox
   is connected, call `connect_mailbox` **with no arguments**. The service
   then asks the user directly, on their own screen, for their address and
   an app password; that form is not part of this conversation and you
   never see what they type. Never ask for a password yourself. Tell the
   user, before calling it, that a form from the mail connector is about to
   appear and why it is separate. If the tool answers that their device is
   being asked to approve the service's folder, tell them to tap it and
   call `connect_mailbox` again. Do not call `request_access` before the
   mailbox is connected: the wallet would answer "nothing to approve".
2. Once `connect_mailbox` has answered linked, call `request_access` for
   the kind and tell the user their device will ask.

An agent whose consent is missing will simply refuse its first run with the
same words you would see; better to settle it now. If a resource is
`declined`, respect that unless the user says otherwise, and only then ask
again with `ask_again: true`.

Do not ask for the assistant's spend consent here; that is settled at
sign-in and is not per agent.

## 4. Tell the user what happens next, precisely

- Within about fifteen seconds the folder is mirrored and the agent exists.
- With `every`, the first run starts within a minute and then on the
  interval; with `on: mail.changes`, at the next arrival.
- The workspace named after the agent appears in the sidebar at its first
  run, with that run as its first session. Sessions are named
  `<Agent name> <date time>`.
- To change the agent, they edit the two files in Drive, or ask you in
  the chat; either way the next run follows the edit. To stop it, set
  `paused: true`.
- To remove it, delete the folder in Drive.

## What an agent is NOT

An agent is two files in the user's Drive. It is **not a program**. Never
write a script, a daemon, a scheduler, a poller or any code for it, never
create files in this workspace for it, and never run anything in the
background yourself: the harness reads `agent.yaml` and runs the agent on
its trigger, in a session of its own, with the same tools you have. If you
find yourself checking for Python, Node or an HTTP API, stop: you have left
this skill. Go back to section 2 and write the two files through Drive's
tools.

## Rules you do not bend

- **Definition changes happen in conversation, never inside a scheduled
  run.** A run that thinks its definition is wrong reports that and stops.
- **One file per fact.** `agent.md` is prose for the model, `agent.yaml` is
  data for the harness; do not duplicate the prompt in both.
- **No secrets in the folder.** A mailbox password is typed into the
  connector's own form, never into an agent file and never into this
  conversation.
- **No new capabilities by prose.** An agent cannot grant itself a resource
  by mentioning it in `agent.md`; only the user's consent, given through
  `request_access`, counts.
- **Do not create an agent for something the chat can do on the spot.**
  A one-off question is a question, not an agent.

## Example: the inbox agent

User: "Triage my inbox as mail comes in, drafts only, and give me a
digest."

Agreed: name **Inbox triage**; runs on `mail.changes`; prompt "Triage what
arrived since the last run following the inbox-triage skill; label, draft
only where a reply is owed, and end with a digest."; it needs
`mail.mailbox`; defaults for the rest. Then section 3 for `mail.mailbox`
(connect first, then `request_access`), and the four sentences of section 4.
