# Customer API

Everything the dashboard does for your organization, you can do over HTTP with an API key. This guide covers the parts that aren't obvious from the spec: what a key is, how lists and errors look, how runs work over time, and which steps still need a person.

The spec is the source of truth for exact paths, fields and codes. This guide is the map around it.

- Base URL: `https://app.dozaldevs.com`
- Spec: `https://app.dozaldevs.com/openapi.json` (no key needed to read it)

The spec covers the routes a customer or a key can reach, and it's built from the same route declarations the server uses, so the two can't drift apart. Every operation lists the minimum level it needs and says whether a key may call it at all.

## Your key

An API key belongs to exactly one organization. It starts with `dze_` and you send it as a bearer credential. Agents that have the Dozal skills use the `dozaldevs` command line, and the `curl` examples in this guide are for callers without the CLI.

```bash
curl -s https://app.dozaldevs.com/me/tenants \
  -H "Authorization: Bearer $KEY"
```

A key sees its own organization and nothing else. Ask for another organization's id and you get a `404`, the same answer you'd get for an id that doesn't exist, so a key can't be used to find out what exists elsewhere. `GET /me/tenants` is the easy way to learn the organization id your key belongs to. You'll put that id in most paths as `:tenantId`.

### The four levels

A person who owns the organization picks a level when they create the key. The key then acts like a member of that level, using the same checks a signed-in person goes through.

- Viewer can read: Dozals, runs, run logs, connections, members, and the organization's handoff map.
- Member can also do things: start a run, approve or reject a run's output, start connecting an app, create and change handoffs.
- Admin can also hire and edit Dozals (the Hiring Dozal chat lives under `/admin/tenants/:tenantId/hiring/conversations`), and can list the organization's keys.
- Owner can also change organization settings, work through onboarding, and open remote-browser sign-ins.

A route that needs a higher level than your key has answers `403` with `FORBIDDEN`. Keys created before levels existed read as Admin.

### What no key can do

Some actions are closed to a key of every level, Owner included, because a leaked key could otherwise lock the owner out or hand the organization away. They answer `403` with `API_KEY_NOT_ALLOWED`:

- creating or revoking an API key
- inviting someone, changing a member's role, removing a member
- deleting or restoring the organization, or suspending it with a status change
- accepting or declining a billing attach request

Other routes are about one person, not an organization, so they have no meaning for a key. They answer `403` with `PERSON_ONLY`:

- a person's notifications and their read state
- push subscriptions and notification preferences (`/me/push-subscriptions`, `/me/notification-preferences`)
- creating an organization, accepting an invitation, and the person's own list of waiting invitations
- billing accounts (`/admin/billing-accounts/...`)
- progression and its celebrations, and accepting a mission
- recording who is viewing a run
- running the starter Dozal during onboarding, which emails the signed-in person
- GitHub's "link an installation that already exists" picker

Every operation in the spec carries an `x-auth` entry that says which of these applies, so an agent can check before it calls.

## From the command line

Every operation in this guide is also a `dozaldevs` command, and the CLI reads the live spec each time (the operate skill has the setup). Run `dozaldevs <command> --help` to see one operation's ids, options, request body and responses. The paths and fields below are the same either way, so what you read here carries over to the commands.

## Staying up to date

The CLI names itself on every call with `X-DozalDevs-Client: cli/<version> skills/<version>`. Every call made with a key answers with `X-DozalDevs-Skills-Latest` (the newest published skills) and `X-DozalDevs-CLI-Minimum` (the CLI those skills need). When the client is behind, the answer also carries `X-DozalDevs-Notice` and a `notices` list in the JSON body, one plain line each, like `DozalDevs skills update available (2026.10.27-abc1234): run npx skills update`. A call that sends a key but no `X-DozalDevs-Client` is treated as an old CLI and told to update both. The CLI reads the skills version from the installed skills itself (the nearest `.agents/skills`, then `~/.agents/skills`). MCP calls get no notices: the MCP server's instructions are always current. An old version never makes a call fail.

`GET /client-versions` needs no key. It returns the latest skills version with a one-line note per version, the CLI minimum, and the notices for the `X-DozalDevs-Client` you send. `dozaldevs version` prints the same in plain words.

## Lists

Every list route returns the same envelope:

```json
{
  "items": [],
  "page": { "next_cursor": null, "limit": 50, "has_more": false }
}
```

If `has_more` is true, pass `page.next_cursor` back as `?cursor=` to get the next page, and keep every other query parameter the same. The cursor is opaque. Don't build one or take one apart.

Lists accept `limit`, `cursor` and `sort` (written `field.asc` or `field.desc`, for example `created_at.desc`). The allowed range for `limit` is in the spec for each list, and a value outside it is clamped to the nearest end instead of rejected. A cursor that doesn't belong to the query, or a sort field the list doesn't offer, answers `400` with `INVALID_CURSOR` or `INVALID_SORT`.

Counting the items you were handed is the wrong way to count a list, because `items` stops at `limit`. Add `?include_total=true` and read `page.total`. It makes the server run an extra count, so it's off unless you ask. The value has to be the literal word `true`.

```bash
curl -s "https://app.dozaldevs.com/admin/tenants/$TENANT/tasks?limit=20&include_total=true" \
  -H "Authorization: Bearer $KEY"
```

## Errors

A failed request returns JSON with the same fields every time:

```json
{
  "error": "NOT_FOUND",
  "message": "optional detail",
  "next_step": "List the collection to find a current id that belongs to this organization, then retry with that id."
}
```

`error` is a code you can branch on. `message` is extra detail and isn't always there. `next_step` is one plain sentence about what to do next, and it's always there. A `400` can also carry `issues`, which names the fields that were wrong, and a `500` may carry a `reference_id` worth quoting to support. The spec lists every code in the `ErrorCode` schema.

The codes you'll meet most:

- `401` with `AUTHENTICATION_REQUIRED` or `INVALID_TOKEN`: no key was sent, or it's wrong, revoked or expired.
- `403` with `FORBIDDEN`: the key's level is too low for this route.
- `403` with `API_KEY_NOT_ALLOWED` or `PERSON_ONLY`: see the two lists above. A higher level won't help.
- `404` with `NOT_FOUND`: the id doesn't exist in your organization, or it belongs to a different one.
- `400` with `INVALID_REQUEST` or `INVALID_ID`: the request didn't match the schema. Ids must be UUIDs.

A path no route claims gets a plainer answer. It's a `404` with just `{ "error": "Not Found" }` and no `next_step`. If you see that body, check the path against the spec before you check anything else.

## Runs take time

Starting a run doesn't wait for it to finish. The call returns right away with an id, and the work happens afterward, sometimes a few seconds later and sometimes much longer, and sometimes it stops partway to wait for a person to approve something.

```bash
curl -s -X POST \
  "https://app.dozaldevs.com/admin/tenants/$TENANT/employees/$SLUG/trigger" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{ "inputs": { "topic": "weekly summary" } }'
```

You get `202` back with `task_id` and `status_url`. Keep the `task_id`. Add `?dry_run=true` to find out what would start without starting anything.

Then watch the run. `GET /admin/tenants/:tenantId/tasks/:taskId` returns the task and a `phase_category` that boils its status down to three values:

- `working`: still going. This includes a run that was approved or rejected a moment ago and is still being finished.
- `awaiting_human`: the output is waiting for someone to approve or reject it.
- `terminal`: the run ended, as `Done`, `Failed` or `Cancelled`.

Poll that route every few seconds, or use the live stream below. When `phase_category` is `awaiting_human`, a Member-level key can settle it:

```bash
curl -s -X POST \
  "https://app.dozaldevs.com/admin/tenants/$TENANT/tasks/$TASK/decision" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{ "action": "approve" }'
```

`action` is `approve` or `reject`, and `reason` is optional. It's the same decision the buttons in Slack record, and the dashboard shows it as made by your key's name.

To see what's waiting, `GET /admin/tenants/:tenantId/tasks?needs_decision=true` filters the list, and `GET .../tasks/:taskId/pending-approval` lists the requests still open on one run.

### Watching live

Two routes stream while a run is in progress. Both use server-sent events, and `curl -N` turns off buffering so you see each event as it arrives.

```bash
curl -N "https://app.dozaldevs.com/admin/tenants/$TENANT/tasks/$TASK/status/stream" \
  -H "Authorization: Bearer $KEY"
```

The status stream sends an `event: status` line each time something changes, with `status` and `phase_category` in the data, and ends with `event: done` once the run is terminal. The sibling `.../tasks/:id/progress/stream` sends `event: progress` for each step the run takes. It stops with `event: done` when the run finishes, or after about fifteen minutes if the run is still going. In that case the `done` event says so, and you can open the stream again.

### One run per line

Some Dozals take a list as an input and start one run for each line. The trigger response for those has more in it: `task_ids` for every run that started, `runs` with each line's own `task_id` and `status_url`, and `failures` for any line that didn't start. `task_id` and `status_url` at the top are just the first run's. A list over the Dozal's limit is refused up front with `422` and `TOO_MANY_ITEMS`, and `?dry_run=true` returns a `run_count` so you can check first.

A Dozal only does this when its input list is set up for it. The input has to be an `every_run` input of type `long_text` with `one_run_per_line` set to `true`, and only one input on a Dozal can carry it. A second one is refused with `400` and `INVALID_REQUEST`. Blank lines and repeated lines are dropped, and each remaining line becomes the value of that input for its own run.

You set it through the Hiring Dozal's edit chat, or by sending `PATCH /admin/tenants/:tenantId/archetypes/:archetypeId` with an `input_schema` that includes the flag. `input_schema` replaces the whole list, so send every input the Dozal has, not just the one you're changing. A trimmed example of the item:

```json
{
  "input_schema": [
    {
      "key": "addresses",
      "label": "Addresses",
      "type": "long_text",
      "frequency": "every_run",
      "required": true,
      "one_run_per_line": true
    }
  ]
}
```

### Starting a run from your own system

To start a Dozal from a cron job, CI or another service, call the same trigger route with a key minted for that system. Member level is enough to run Dozals and settle approvals, and naming the key after the system tells you later which runs it started.

- Find the Dozal's `slug` with `GET /admin/tenants/:tenantId/archetypes` (the Dozal list). The slug is stable; the display name isn't.
- Run it once with `?dry_run=true` first. A missing required input answers `422` with `MISSING_REQUIRED_INPUTS` and lists them in `missing`, so you learn the input keys without starting anything.
- `inputs` takes string values keyed by the Dozal's input keys. `prompt` is an optional free-text note for this one run.
- Keep the `task_id` from the `202`, then poll `GET .../tasks/:taskId` or open the status stream until `phase_category` is `terminal`, and read `output` from the task.

What isn't there yet, so you can plan around it:

- There's no idempotency key. Every accepted call starts a new run, so if your client retries after a timeout, list `GET .../tasks?archetype_id=:archetypeId&since=...` first to see whether the first call already started one.
- There's no per-Dozal webhook URL to receive your events, and no callback when a run finishes. Your system calls the trigger route and then polls or streams. A Dozal can also start on its own from a schedule, from an event in a connected app, or from another Dozal finishing (see Handoffs below).

## The run's logs

Once a run has started, the API can show you what it recorded. We call the whole set the run's logs. They're all readable at Viewer level, and these are the pieces:

- `GET /admin/tenants/:tenantId/tasks/:taskId/events` is the structured one, and the place to start.
- `GET .../tasks/:taskId/conversation` returns ad-hoc conversations only: the messages a sender wrote and the replies that went back to them. For a Dozal run it's empty (`items` is `[]`), so don't read an empty answer as "nothing happened". What a Dozal was asked and what it sent live in the task's `output`, in its `events` and in its `executions`.
- `GET /admin/tenants/:tenantId/executions?task_id=:taskId` lists each attempt at the run. Add `?id=` for one attempt in full, including its transcript.
- `GET .../tasks/:taskId/status-log` is every status change the task went through, oldest first by default.
- The progress stream above, if you're watching while it happens.

### What the events contain

The `events` response is one JSON object for the run. It holds:

- `phases`: each phase of the run, with every tool call in it. A call has its name, input, output, whether it errored, its exit code (for shell commands), when it started and ended, how long it took, and `retry_of_failed_call`, which marks a call that repeats one that just failed.
- `retried_failures`: the calls that failed and were tried again, with the error and whether the retry worked.
- `approval_waits`: each time the run stopped for a person, when the wait began and ended, and whether a decision, a timeout, a failure or completion ended it.
- `status_timeline`: each status change with how long the run stayed in the previous one.
- `tool_ledger` and `app_calls`: the platform's own record of tool calls and of calls to connected apps, with success, status code and error.
- `run_inputs`, `failure_code`, `failure_reason` and `outcome_code`.
- `config_at_run`: the Dozal's instructions and setup as they stood when the run began, and `edits_since_run`, which lists what has been changed since.

Text inside these fields is the tools' own text, and known secret values are masked out of it. Long tool output is shortened to its beginning and its end.

### How to read them, and what they can't promise

Read `status`, `failure_code` and `failure_reason` first (the reason is the run's own account of why it stopped), then `status_timeline` to see where the run was when it ended. Look at `retried_failures` and at any phase whose calls have `is_error` set. Check `approval_waits` if the run seems stalled, since a run waiting on a person is working as designed and not stuck. Then read `output` on the task and the transcript in `executions` for what the Dozal was asked and what it sent.

That gives you a detailed record of what happened. It doesn't hand you a verdict. We tested it. We took eight real production runs that had gone wrong and whose causes we already knew, gave a fresh agent only what this API returns plus the customer's complaint, and graded its answers blind. It named the real cause for five of the eight. So treat the logs as evidence to reason from. They often point at the answer, and sometimes they don't. For instance, they don't record which of a run's inputs actually reached the Dozal, so a missing input can look like nothing at all. When the logs leave you unsure, say so, and don't guess a cause.

## Hiring a Dozal

Hiring and editing happen in a chat with the Hiring Dozal, and every route is Admin level. The chats live under `/admin/tenants/:tenantId/hiring/conversations`.

1. Open a chat with `POST .../hiring/conversations`. The body is `{ "kind": "create" }` for a new Dozal, or `{ "kind": "edit", "archetype_id": "..." }` to change an existing one. If that Dozal already has an open chat, you get it back instead of a new one.
2. Send a message with `POST .../hiring/conversations/:conversationId/messages`. The body takes `message`, `answers` (when you're answering its questions, keyed by question id) and `turn_seq`, which must be the chat's current `turn_seq`. It returns `202` with `message_id`, `event_seq`, `turn_seq` and `machine`. The reply isn't in that response.
3. Read the chat with `GET .../hiring/conversations/:conversationId` until `turn_in_progress` is `false`. The new reply is the last entry in `messages`, and any questions it needs answered are in `open_questions`.
4. `presented` turns `true` once the Hiring Dozal has offered its draft, and `summary` then says what it built. When that looks right, `POST .../hire` (new Dozal) or `POST .../apply` (edit). Send `{ "turn_seq": N }` with the current value so a stale view can't save the wrong draft. An edit whose draft differs from nothing in the live Dozal saves nothing: `apply` answers `200` with the Dozal as it is, the version stays and no history row is written.

The two calls that return a chat don't have the same shape, and that trips people up. `GET .../hiring/conversations/:conversationId` returns the chat's fields at the top level:

```json
{
  "id": "6f1c0f0e-5b0e-4a3c-9d57-0d3b2c1a9e11",
  "kind": "create",
  "status": "open",
  "closed_reason": null,
  "archetype_id": "b2d5a4c8-3f1e-4c0a-8a55-2f7d9e6b1c33",
  "turn_seq": 2,
  "turn_in_progress": false,
  "messages": [],
  "open_questions": [],
  "draft": null,
  "presented": false
}
```

`POST .../hiring/conversations` returns the same fields inside `conversation`, next to a `created` flag. The status is `201` when a new chat was made and `200` when you got an open one back:

```json
{
  "created": true,
  "conversation": {
    "id": "6f1c0f0e-5b0e-4a3c-9d57-0d3b2c1a9e11",
    "kind": "create",
    "turn_seq": 0,
    "turn_in_progress": false,
    "messages": [],
    "open_questions": [],
    "draft": null,
    "presented": false
  }
}
```

Both examples are trimmed. The real chat also has `last_event_seq`, `summary`, `context_full`, `approval_off_confirmed`, `sites_needing_sign_in`, `recording` and `saved_dozal`, and the spec lists them all. `POST .../restart` answers `201` with `{ "conversation": { ... } }` and no `created`. The chat's `archetype_id` is the id of the Dozal being drafted or edited, so after a hire it's the id of the new Dozal.

Other chat calls: `.../stop` ends a reply that's still running, `.../discard` throws the chat away, and `.../restart` closes it and opens a fresh one on the same Dozal with the draft carried over. A message sent while a reply is still running is refused with `TURN_IN_PROGRESS`, and one sent to a closed chat with `CONVERSATION_CLOSED`.

## Handoffs

A handoff makes one Dozal start another when it finishes. The API calls them subscriptions.

```bash
curl -s -X POST \
  "https://app.dozaldevs.com/admin/tenants/$TENANT/subscriptions" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{ "upstream_archetype_id": "'"$FIRST"'", "downstream_archetype_id": "'"$SECOND"'" }'
```

A Member-level key can create one. `PATCH` and `DELETE` on `/admin/tenants/:tenantId/subscriptions/:id` switch one off or on, set `propagate_no_action` (whether the chain continues after a run that found nothing to do), or remove it. To see the handoffs around one Dozal, use `GET .../archetypes/:archetypeId/subscriptions` (Member level). `GET .../subscription-graph` returns the whole organization's map, and Viewer can read it. A run that was started by a handoff shows its place in the chain in the `graph` field of its task.

## Connecting apps

Connecting Slack, GitHub or another app is an OAuth flow, and the last step is always a person clicking Allow on the other service's own screen. A key can start the flow but can't finish it. The start call returns a link, and your job is to hand that link to a person.

- Slack: `POST /admin/tenants/:tenantId/slack/install/start` returns `{ "url": "..." }`.
- GitHub: `POST /admin/tenants/:tenantId/github/install/start` returns `{ "url": "..." }`.
- Other apps: the connect route under connections in the spec takes the app's `toolkit` name as a query parameter and returns `{ "url": "..." }`. The `url` is `null` when the app can't be connected that way.

All three need Member level. Once the person has clicked through, check `list-app-connections` (`GET /admin/tenants/:tenantId/composio/connections`) to see an app show up, or `GET /admin/tenants/:tenantId/integrations` for Slack and GitHub, and `GET .../integrations/attention` for connections that have stopped working. Apps like Gmail or Jira are never in `/integrations`, so don't read their absence there as "not connected".

GitHub has one extra case. If the installation already exists and the person who authorized it can't see it, the callback answers `409` with `GITHUB_LINK_NEEDS_PERSON`. The agent will see GitHub still unconnected in the list. A signed-in person has to link it from Integrations in the dashboard.

## Using your connected apps directly

You don't need a Dozal to use an app the organization already connected. To read or change something in a connected app, run the action directly with `run-app-action` (`run_app_action` over MCP); never create a Dozal just to run one action. Your key can run an action on it, and DozalDevs makes the call with the organization's connection, so you never see the app's password or token.

```bash
dozaldevs list-app-connections                        # what's connected
dozaldevs list-app-actions googlesheets               # its actions, and whether your key may run each
dozaldevs get-app-action googlesheets GOOGLESHEETS_BATCH_GET   # one action's inputs
dozaldevs run-app-action googlesheets GOOGLESHEETS_BATCH_GET --data arguments.spreadsheet_id=abc --data 'arguments.ranges:=["A1:C10"]'
```

Send one `session_id` of your own with every call that belongs to the same job, such as your conversation id: `--data session_id=<your id>` (saved HTTP endpoints take it too). The organization then sees that job's calls together as one session. Without it, your key's calls group by day (UTC).

The app's inputs go under `arguments`. The answer is `{ "successful": true|false, "data": ..., "error": ..., "provider_status": 200 }`. `successful: false` means the app itself said no (a bad id, a missing sheet), so read `error` and fix the inputs.

What your key may run depends on its level and on the action:

- A Viewer key runs only actions that read.
- Member and above run any action, including the ones that pay or refund money, delete data, cancel something, change an access code or password, remove a person's access, or publish something publicly, and actions DozalDevs hasn't classified yet. Nothing waits for approval and nothing is held, so the human gives a key that level only when they trust it with those actions. The action's kind stays on the call in Activity as `hold_kind`.
- `list-app-actions` shows `allowed_for_you` for each action. For a Viewer key, `null` means the action hasn't been checked yet and is checked the first time someone runs it.

Each call is charged as normal usage and shows on the Billing page as "Direct app calls". Owners and admins see every call, with the key that made it, under "Calls to your apps" on the API Keys page (`list-app-calls`). A call DozalDevs refused (not allowed for your key's level, or the app not connected) is listed too, with `result: "not_run"` and the reason, and is not charged. There are two limits:

- An organization can make a limited number of direct calls a minute. Past it you get `429` with `RATE_LIMITED` and a `Retry-After` header.
- When the platform's monthly app-call allowance runs short, direct calls pause for everyone with `503` and `DIRECT_APP_BUDGET_REACHED`, so Dozals keep working. Try again later.

## What still needs a person

An agent can do almost everything, but a few steps are a human's by design:

- Creating the organization. `POST /me/organizations` is person-only, so the first sign-up and the organization it makes happen in a browser.
- The first key. A key can't create another key, so a signed-in owner makes the first one under Setup, API Keys in the dashboard's left menu, picks its level, and copies it. The secret is shown once.
- Consent clicks. Every OAuth connect ends with a person pressing Allow at the other service.
- Remote-browser sign-in. For apps that need a password typed into a real page, `POST /admin/tenants/:tenantId/browser-sessions/start` opens a live browser, and a person signs in there. The same goes for the Airbnb sign-in routes under `/airbnb/login`. An Owner-level key can open these sessions, but it can't type the password.
- GitHub's link-an-existing-installation picker, as above.

Everything else, from hiring and running to approving and watching, goes through the routes in the spec. When you aren't sure whether a route is open to your key, read its `x-auth` first.
