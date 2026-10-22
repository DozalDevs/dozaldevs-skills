---
name: hire-ai-employee
description: "Use when a human wants to create, hire, activate, or set up a new AI employee from a plain-English description, e.g. wanting something that writes a weekly recap or replies to guest messages. Drives the Hiring Dozal's multi-turn setup wizard (the hiring chat), relays its button questions (show me first or just send it) to the human, and hires the finished Dozal. Use it for any new-employee request. Load operate-ai-employees first for auth and base-URL setup."
---

# Hire an AI Employee

This sets up a new AI employee (a Dozal) by chatting with the **Hiring Dozal**, the platform's own setup assistant. It asks questions, looks things up (which apps are connected, which Slack channel you mean), shows a finished Dozal, and you hire it.

> Prerequisite: load `operate-ai-employees` first. It connects Restish and sets `AI_EMPLOYEE_API_KEY` and `AI_EMPLOYEE_TENANT_ID`. The chat needs a key at **Admin** level or above; a lower key answers `403 FORBIDDEN`.

Concepts and response shapes: `../operate-ai-employees/references/customer-api.md`. For one command's arguments, run `restish dozaldevs <command> --help`. This skill keeps only how the chat behaves.

## How the chat works

The chat lives on the server. You open it once, then send one message at a time. Each send answers `202` straight away and the reply is written later (up to about a minute and a half when the first machine has to start). So every turn is: send, wait, read.

1. **Open.** Keep the chat id.

```sh
CHAT=$(restish dozaldevs start-hiring-chat "$AI_EMPLOYEE_TENANT_ID" -f body.conversation.id -o lines <<'EOF'
{"kind":"create"}
EOF
)
```

2. **Send.** Read the current `turn_seq` first, and send it back with the message.

```sh
SEQ=$(restish dozaldevs get-hiring-chat "$AI_EMPLOYEE_TENANT_ID" "$CHAT" -f body.turn_seq -o lines </dev/null)
restish dozaldevs send-hiring-message "$AI_EMPLOYEE_TENANT_ID" "$CHAT" <<EOF
{"message":"$MESSAGE","turn_seq":$SEQ}
EOF
```

3. **Wait.** Run this, and run it again while it prints `true`. It stops on `false` twice in a row, or after 80 seconds. The turn worked when `turn_seq` went up by one. If not, the last `messages` entry with `error:true` says why.

```sh
end=$(( $(date +%s) + 80 )); prev=; t=; while [ $(date +%s) -lt $end ]; do t=$(restish dozaldevs get-hiring-chat "$AI_EMPLOYEE_TENANT_ID" "$CHAT" -f body.turn_in_progress -o lines </dev/null) || { restish dozaldevs get-hiring-chat "$AI_EMPLOYEE_TENANT_ID" "$CHAT" </dev/null; break; }; [ "$t" = false ] && [ "$prev" = false ] && break; prev=$t; sleep 3; done; echo "$t"
```

4. **Read.** Run `restish dozaldevs get-hiring-chat "$AI_EMPLOYEE_TENANT_ID" "$CHAT" </dev/null`. Show the human the new assistant entries in `body.messages` and every entry in `body.open_questions`.
5. **Answer.** Send the next message with the current `turn_seq`. Button presses go in `answers`, keyed by the question's `id`: `{"q1":{"option":"approval_on"}}`, or `{"q2":{"text":"..."}}` where typing is allowed. `message` may be empty when you only press buttons.

```sh
restish dozaldevs send-hiring-message "$AI_EMPLOYEE_TENANT_ID" "$CHAT" <<EOF
{"message":"","turn_seq":$SEQ,"answers":{"q1":{"option":"$OPTION"}}}
EOF
```

6. **Stop when ready:** `body.presented` is `true`, `body.open_questions` is empty and `body.draft` is set.

A long, specific first message (when it runs, what it produces, where results go, which apps) usually gets straight to a finished Dozal. "Make me a bot" always gets questions. Keep going until it's ready; stopping after the first question is the most common mistake.

## The three questions the platform asks itself

These carry a `protocol` field and must be answered with a **button**. Typed text never changes them.

| `protocol`         | What it's asking                                                                   | Option values                                                |
| ------------------ | ---------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `approval`         | "Show you each result before it goes out, or just send it?"                        | `approval_on` (show me first), `approval_off` (just send it) |
| `schedule`         | An app can't announce new items at once, so the Dozal will check every 15 minutes. | `schedule_ok`, `schedule_talk`                               |
| `changes_each_run` | Will the human give this value each time, or should it always be the same?         | `each_time`, `always_same`                                   |

Relay each one in plain words and send the human's own choice. **Never press "just send it" for them**, even to finish faster.

## When a send is refused

| Code                            | What to do                                                                                                                       |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `409 TURN_IN_PROGRESS`          | Wait for the turn to finish, then send.                                                                                          |
| `409 DRAFT_CONTINUED_ELSEWHERE` | Someone moved the chat on. Read it back, show what changed, resend once with the new `turn_seq` if the message still makes sense. |
| `503 HIRING_BUSY`               | No setup machine yet. Wait a minute and send again.                                                                              |
| `409 CONTEXT_FULL`              | The chat got long. Restart it (below) for a fresh one on the same draft.                                                         |
| `409 CONVERSATION_CLOSED`       | It already finished (`details.closed_reason`). Open a new one.                                                                   |
| `503 HIRING_UNAVAILABLE`        | Setup by chat is off right now. Tell the human and try later.                                                                    |

```sh
restish dozaldevs restart-hiring-chat "$AI_EMPLOYEE_TENANT_ID" "$CHAT" <<'EOF'
{}
EOF
```

## Hiring the Dozal

Show the human a plain-English summary from `body.draft` (name, what it does, when it runs, where results go, whether it asks before sending) and confirm. On "yes", send the chat's current `turn_seq` (read it as in step 2):

```sh
restish dozaldevs hire-from-hiring-chat "$AI_EMPLOYEE_TENANT_ID" "$CHAT" <<EOF
{"turn_seq":$SEQ}
EOF
```

`200` means it's live and the chat closes. A refusal comes in plain words (for example `400 MISSING_CONNECTION` when an app still has to be connected). Tell the human what to fix, then keep chatting or hire again. If the missing piece is a connection, starting it returns a link only a person can open (see `operate-ai-employees`).

To throw the setup away:

```sh
restish dozaldevs discard-hiring-chat "$AI_EMPLOYEE_TENANT_ID" "$CHAT" <<'EOF'
{}
EOF
```

The hired Dozal's id is `body.archetype_id` on the chat. Its slug is the stable name you run it by:

```sh
DOZAL=$(restish dozaldevs get-hiring-chat "$AI_EMPLOYEE_TENANT_ID" "$CHAT" -f body.archetype_id -o lines </dev/null)
restish dozaldevs list-dozals "$AI_EMPLOYEE_TENANT_ID" -q id="$DOZAL" -f 'body.items[0].slug' -o lines </dev/null
```

## After hiring

Tell the human it's live and offer to run it: "It's set up! Want me to run it now so you can see what it does?" Then load `run-ai-employee`.
