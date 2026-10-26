---
name: manage-ai-employees
description: "Use when a human wants to see, list, pause, reactivate, rename or inspect their existing AI employees (config, status active/paused/draft), or manage connected apps, schedules, handoffs, members and invitations, usage and bills, e.g. what employees do I have, pause the recap bot, how did last Friday go. Not for running one now (run-ai-employee), a task's output, approval or logs (monitor-and-review-ai-employee-work), or creating one (hire-ai-employee). Load operate-ai-employees first."
---

# Manage AI Employees

This lists and manages the human's existing AI employees and the account around them.

> Prerequisite: load `operate-ai-employees` first. It sets up `restish dozaldevs` and `AI_EMPLOYEE_TENANT_ID`, and explains levels, paging and errors.

Every call below passes `"$AI_EMPLOYEE_TENANT_ID"` first. For a command's arguments and body, run `restish dozaldevs <command> --help`. Error codes and concepts: `../operate-ai-employees/references/customer-api.md`. This file is only the behavior you can't read off the spec.

## List employees

```sh
restish dozaldevs list-dozals "$AI_EMPLOYEE_TENANT_ID" </dev/null
```

The answer is `{ "items": [...], "page": { "next_cursor", "limit", "has_more" } }`. While `page.next_cursor` isn't null, repeat with `-q cursor=<next_cursor>`. Useful fields:

- `role_name`: the display name (renameable).
- `slug`: the stable identifier `run-ai-employee` needs. It never changes, even after a rename.
- `status`: `active` (live), `paused` (was live, temporarily off), `draft` (saved, never activated), `superseded` (replaced).
- `identity`, `overview`: what it is, in plain language.

Present a friendly roster, not raw JSON: "You have 2 employees: **Weekly Recap Writer** (active, runs Fridays) and **Guest Responder** (active)."

## Pause, reactivate, rename, edit

`update-dozal` takes the archetype UUID from the list (not the name) and needs **Admin**. Pausing keeps its "was live" identity, and a paused employee won't run, even on its schedule. Use `paused` for a live employee, never `draft`.

```sh
restish dozaldevs update-dozal "$AI_EMPLOYEE_TENANT_ID" "$DOZAL" <<'EOF'
{"status":"paused"}
EOF
```

Send `{"status":"active"}` to reactivate; it re-checks connections.

Each content edit bumps the Dozal's `version` and must send the current one as `base_version` (else `400 BASE_VERSION_REQUIRED`):

```sh
restish dozaldevs list-dozals "$AI_EMPLOYEE_TENANT_ID" -q id="$DOZAL" -f 'body.items[0].version' -o lines </dev/null
restish dozaldevs update-dozal "$AI_EMPLOYEE_TENANT_ID" "$DOZAL" <<EOF
{"role_name":"$NEW_NAME","base_version":$VERSION}
EOF
```

If someone changed it since, `409 VERSION_CONFLICT`: read again and redo your change. Reverts and template updates need `base_version` too; a status-only update needs none. Approving or denying a suggested cleanup needs no version; `409 SUGGESTION_OUT_OF_DATE` means it was withdrawn: re-read, don't retry.

Each save answers `changes`, what really changed; trust it over a re-read. Test an update first: `-q validate_only=true`. Fields only `update-dozal` changes, seeing what it's told, undoing: `references/checking-and-undoing.md`.

To change what the employee actually does ("make the summary shorter", "also post to Slack"), use the Hiring Dozal's edit chat described in `hire-ai-employee`:

```sh
restish dozaldevs start-hiring-chat "$AI_EMPLOYEE_TENANT_ID" <<EOF
{"kind":"edit","archetype_id":"$DOZAL"}
EOF
```

When the chat shows the finished change (`presented` true, `open_questions` empty), show the human what will change and, on "yes", save it with the chat's current `turn_seq`. Nothing changes on the live employee until then.

```sh
restish dozaldevs apply-hiring-chat "$AI_EMPLOYEE_TENANT_ID" "$CHAT" <<EOF
{"turn_seq":$TURN_SEQ}
EOF
```

`409 VERSION_CONFLICT` on apply means it changed elsewhere: run `restart-hiring-chat` with `{}` to pick that up.

## Connections

```sh
restish dozaldevs list-integrations "$AI_EMPLOYEE_TENANT_ID" </dev/null
restish dozaldevs list-integrations-needing-attention "$AI_EMPLOYEE_TENANT_ID" </dev/null
restish dozaldevs list-app-connections "$AI_EMPLOYEE_TENANT_ID" </dev/null
```

In order: what's connected, what needs fixing, and apps with their health.

Connecting is two steps and the second is a person's. Start it, which returns a `url`. Give it to the human, who opens it and approves. Then run `list-integrations` to see it connected. Don't claim it worked before you've seen that.

```sh
restish dozaldevs start-app-connection "$AI_EMPLOYEE_TENANT_ID" -q toolkit="$APP" -f body.url -o lines </dev/null
restish dozaldevs start-slack-install "$AI_EMPLOYEE_TENANT_ID" <<'EOF'
{}
EOF
restish dozaldevs start-github-install "$AI_EMPLOYEE_TENANT_ID" <<'EOF'
{}
EOF
```

GitHub's "link an installation that already exists" is a person's job (`403 PERSON_ONLY`). Confirm before disconnecting anything, because employees that use it stop working.

## Schedules

```sh
restish dozaldevs list-schedules "$AI_EMPLOYEE_TENANT_ID" </dev/null
restish dozaldevs get-dozal-schedule "$AI_EMPLOYEE_TENANT_ID" "$DOZAL" </dev/null
```

In order: scheduled employees with their next run, and one employee's next and last run. To change the timing, edit the employee as above. Both of these send `{}`: `run-schedule-now` (it takes the employee's `$SLUG`) makes the next scheduled run start now, and `resume-dozal-schedule` switches a scheduled employee back on after repeated failures paused it. Find out why it failed first (`monitor-and-review-ai-employee-work`).

```sh
restish dozaldevs run-schedule-now "$AI_EMPLOYEE_TENANT_ID" "$SLUG" <<'EOF'
{}
EOF
restish dozaldevs resume-dozal-schedule "$AI_EMPLOYEE_TENANT_ID" "$DOZAL" <<'EOF'
{}
EOF
```

## Handoffs

A handoff makes one employee start another when it finishes. `get-handoff-graph` shows the whole workflow, `list-dozal-handoffs` one employee's. Say plainly which employee starts which.

```sh
restish dozaldevs get-handoff-graph "$AI_EMPLOYEE_TENANT_ID" </dev/null
restish dozaldevs list-dozal-handoffs "$AI_EMPLOYEE_TENANT_ID" "$DOZAL" </dev/null
restish dozaldevs create-handoff "$AI_EMPLOYEE_TENANT_ID" <<EOF
{"upstream_archetype_id":"$UPSTREAM","downstream_archetype_id":"$DOWNSTREAM"}
EOF
restish dozaldevs update-handoff "$AI_EMPLOYEE_TENANT_ID" "$HANDOFF" <<'EOF'
{"status":"inactive"}
EOF
restish dozaldevs delete-handoff "$AI_EMPLOYEE_TENANT_ID" "$HANDOFF" </dev/null
```

`update-handoff` turns one `active` or `inactive`; `delete-handoff` removes it.

## Members and invitations

```sh
restish dozaldevs list-members "$AI_EMPLOYEE_TENANT_ID" </dev/null
restish dozaldevs list-invitations "$AI_EMPLOYEE_TENANT_ID" </dev/null
restish dozaldevs revoke-invitation "$AI_EMPLOYEE_TENANT_ID" "$INVITATION" <<'EOF'
{}
EOF
```

`revoke-invitation` cancels a pending one. A key can't invite anyone or change or remove a member (`403 API_KEY_NOT_ALLOWED`): tell the human to do that in the dashboard.

## Usage and bills

Read only. `get-cost` shows what the organization was charged over a date range (`-q from=... -q to=...`), `list-run-costs` the runs with what each cost, `get-run-cost` one run, `get-billing` the balance and status. Report figures as given. Paying, buying credits, billing accounts and moving the organization between accounts are a person's (`PERSON_ONLY` or `API_KEY_NOT_ALLOWED`).

```sh
restish dozaldevs get-cost "$AI_EMPLOYEE_TENANT_ID" -q from="$FROM" -q to="$TO" </dev/null
restish dozaldevs list-run-costs "$AI_EMPLOYEE_TENANT_ID" -q from="$FROM" -q to="$TO" </dev/null
restish dozaldevs get-run-cost "$AI_EMPLOYEE_TENANT_ID" "$TASK" </dev/null
restish dozaldevs get-billing "$AI_EMPLOYEE_TENANT_ID" </dev/null
```

## Review recent runs

```sh
restish dozaldevs list-tasks "$AI_EMPLOYEE_TENANT_ID" -q limit=10 </dev/null
```

It returns the same `items` and `page` envelope. For one run's output or logs, use `monitor-and-review-ai-employee-work` with its `id`.

## Removing an employee

`delete-dozal` is a recoverable soft-delete. Confirm with the human first. Pausing is usually the better first step.

```sh
restish dozaldevs delete-dozal "$AI_EMPLOYEE_TENANT_ID" "$DOZAL" </dev/null
```
