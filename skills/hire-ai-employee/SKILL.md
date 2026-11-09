---
name: hire-ai-employee
description: "Use when a human wants to create, hire, activate, or set up a new AI employee from a plain-English description, e.g. wanting something that writes a weekly recap or replies to guest messages. Drives the Hiring Dozal's multi-turn setup wizard (the hiring chat), relays its button questions (show me first or just send it) to the human, and hires the finished Dozal. Use it for any new-employee request. Load operate-ai-employees first for auth and base-URL setup."
metadata:
  version: "2026.11.09-3ba55bf"
---

# Hire an AI Employee

This sets up a new AI employee (a Dozal) by chatting with the **Hiring Dozal**, the platform's own setup assistant. It asks questions, looks things up (which apps are connected, which Slack channel you mean), shows a finished Dozal, and you hire it.

> Prerequisite: load `operate-ai-employees` first. It installs the `dozaldevs` CLI and explains the key. The chat needs a key at **Admin** level or above; a lower key answers `403 FORBIDDEN`.

Concepts and response shapes: `../operate-ai-employees/references/customer-api.md`. For one command's arguments, run `dozaldevs <command> --help`. This skill keeps only how the chat behaves. Answers are short; `--detailed` gives every field. `--detailed` needs dozaldevs 0.5.0 or newer (an older CLI reads the flag as needing a value).

## What a Dozal can do

Say what the job needs in your own words; the chat picks the matching abilities. Every ability, with when a job needs it, is in `references/abilities.md` (generated from the platform's registry). An organization's own list, with only the apps it connected, is `dozaldevs list-dozal-abilities`.

## How the chat works

The chat lives on the server. You open it once, then send one message at a time. Each send answers `202` straight away and the reply is written later (up to about a minute and a half when the first machine has to start). So every turn is: send, wait, read.

1. **Open.** Keep the chat id.

```sh
CHAT=$(dozaldevs start-hiring-chat --data kind=create --field conversation.id)
```

2. **Send.** Read the current `turn_seq` first, and send it back with the message.

```sh
SEQ=$(dozaldevs get-hiring-chat "$CHAT" --field turn_seq)
dozaldevs send-hiring-message "$CHAT" --data message="$MESSAGE" --data turn_seq:="$SEQ"
```

3. **Wait.** Run this, and run it again while it prints `true`. It stops as soon as the turn ends, or after 80 seconds. The turn worked when `turn_seq` went up by one. If not, the last `messages` entry with `error:true` says why.

```sh
dozaldevs get-hiring-chat "$CHAT" --while turn_in_progress=true --every 3 --timeout 80 --field turn_in_progress
```

4. **Read.** Run `dozaldevs get-hiring-chat "$CHAT"`. Show the human the new assistant entries in `messages` and every entry in `open_questions`.
5. **Answer.** Send the next message with the current `turn_seq`. Button presses go in `answers`, keyed by the question's `id`: `--data answers.q1.option=<value>`, or `--data answers.q2.text="..."` where typing is allowed. `message` may be empty when you only press buttons.

```sh
dozaldevs send-hiring-message "$CHAT" --data message="" --data turn_seq:="$SEQ" --data answers.q1.option="$OPTION"
```

6. **Stop when ready:** `presented` is `true`, `open_questions` is empty and `draft` is set.

A long, specific first message (when it runs, what it produces, where results go, which apps) usually gets straight to a finished Dozal. "Make me a bot" always gets questions. Keep going until it's ready; stopping after the first question is the most common mistake.

## The questions the platform asks itself

The assistant asks in words, and every setting it proposes shows on the change card, where the human keeps or leaves out each part. **Never keep "just send it" for them**, even to finish faster.

Three settings come as button questions about the held change (`protocol` `app_account`, `listen_channel`, `frequent_schedule`): which account of an app the Dozal uses, which Slack channel it listens in for a tag (or `decline`), and whether to keep a schedule that runs more than 24 times a day. Only the human's press counts; the change can't be saved until each is answered. Send the press with no words: words, or "change how often", go to the assistant and turn the change down.

## When a send is refused

| Code                            | What to do                                                                                                                        |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `409 TURN_IN_PROGRESS`          | Wait for the turn to finish, then send.                                                                                           |
| `409 DRAFT_CONTINUED_ELSEWHERE` | Someone moved the chat on. Read it back, show what changed, resend once with the new `turn_seq` if the message still makes sense. |
| `503 HIRING_BUSY`               | No setup machine yet. Wait a minute and send again.                                                                               |
| `409 CONTEXT_FULL`              | The chat got long. Restart it (below) for a fresh one on the same draft.                                                          |
| `409 CONVERSATION_CLOSED`       | It already finished (`details.closed_reason`). Open a new one.                                                                    |
| `503 HIRING_UNAVAILABLE`        | Setup by chat is off right now. Tell the human and try later.                                                                     |

```sh
dozaldevs restart-hiring-chat "$CHAT"
```

## Hiring the Dozal

Nothing saves on its own: the assistant holds a **change to check**, named by the chat's `held_change` (`task_id`, `wait_id`, `action_id`). Read it part by part:

```sh
dozaldevs get-wait-dozal-change "$TASK" "$WAIT" "$ACTION"
```

Show each part's before and after, and the platform's own adjustments (`platform`, with `reason`). Approve all of it, or edit with the part ids to keep (`steps`, `name_and_notes`, `apps_and_tools`, `tool:<path>`, ...), then submit:

```sh
dozaldevs decide-wait-action "$TASK" "$WAIT" --data actionId="$ACTION" --data decision=approve
dozaldevs decide-wait-action "$TASK" "$WAIT" --data actionId="$ACTION" --data decision=edit --data 'keep_sections:=["steps"]'
dozaldevs submit-wait-decisions "$TASK" "$WAIT" --data mode=submit
```

The assistant saves it and reports in the chat, refusals included (a connection still to make). A chat message while a change waits turns it down; the assistant proposes it again if still wanted. `hire-from-hiring-chat` answers `409 USE_THE_CHECK_CARD` with the `held_change`.

To throw the setup away:

```sh
dozaldevs discard-hiring-chat "$CHAT"
```

The hired Dozal's id is `archetype_id` on the chat. Its slug is the stable name you run it by:

```sh
DOZAL=$(dozaldevs get-hiring-chat "$CHAT" --field archetype_id)
dozaldevs list-dozals --id "$DOZAL" --field items[0].slug
```

## After hiring

Tell the human it's live and offer to run it: "It's set up! Want me to run it now?" Then load `run-ai-employee`.
