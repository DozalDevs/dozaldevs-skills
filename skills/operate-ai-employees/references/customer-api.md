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

The CLI prints a short form of each answer, and so does MCP unless you ask for `response_format: "detailed"`. A list keeps each item's main fields, cuts text over 1,000 characters (the cut says "run again with --detailed") and drops empty lists. A single read, such as one Dozal or one draft, is never cut, and `null` stays. `--detailed` gives every field. `--field` always reads the full answer, so it prints a field the short form leaves out. `--all` prints the short form of every page unless you add `--detailed` or `--field`.

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

To count runs per Dozal and status over a period, don't page the list: `GET /admin/tenants/:tenantId/task-counts` answers in one call with `{ total, dozals: [{ archetype_id, role_name, total, by_status }] }`, busiest Dozal first. It takes the list's filters (`since`, `archetype_id`, `role_name`, `status`, `needs_decision`, `approval_required`, `include_test_runs`) and counts exactly the runs `tasks` would list for them. How well the runs went is `GET /admin/tenants/:tenantId/track-record`.

## Errors

A failed request returns JSON with the same fields every time:

```json
{
  "error": "NOT_FOUND",
  "message": "optional detail",
  "request_id": "req_0123456789abcdef0123456789abcdef",
  "next_step": "List the collection to find a current id that belongs to this organization, then retry with that id."
}
```

`error` is a code you can branch on. `message` is extra detail and isn't always there. `request_id` names this one request; the same id is in the `X-Request-Id` header of every response, successes too. Quote it when you report a problem, and cite it in feedback. `next_step` is one plain sentence about what to do next, and it's always there. A `400` can also carry `issues`, which names the fields that were wrong, and a `500` may carry a `reference_id` worth quoting to support. The spec lists every code in the `ErrorCode` schema.

The codes you'll meet most:

- `401` with `AUTHENTICATION_REQUIRED` or `INVALID_TOKEN`: no key was sent, or it's wrong, revoked or expired.
- `403` with `FORBIDDEN`: the key's level is too low for this route.
- `403` with `API_KEY_NOT_ALLOWED` or `PERSON_ONLY`: see the two lists above. A higher level won't help.
- `404` with `NOT_FOUND`: the id doesn't exist in your organization, or it belongs to a different one.
- `400` with `INVALID_REQUEST` or `INVALID_ID`: the request didn't match the schema. Ids must be UUIDs.

A path no route claims gets a plainer answer. It's a `404` with just `error` and `request_id` (`{ "error": "Not Found", "request_id": "req_..." }`) and no `next_step`. If you see that body, check the path against the spec before you check anything else.

A body the server can't read (broken JSON, too large, a bad charset) gets the usual JSON error, `400` or `413` with `INVALID_REQUEST`, never an HTML page.

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

This works for every run that is waiting on a person, however it runs. For a run that holds several actions, `approve` approves all of them and `reject` rejects all of them with your `reason`. To decide one action of such a run, use `decide-wait-action`; `get-wait` shows one wait with the exact content of each action it holds.

To see what's waiting, `GET /admin/tenants/:tenantId/tasks?needs_decision=true` filters the list, and `GET .../tasks/:taskId/pending-approval` lists the requests still open on one run.

### Watching live

Two routes stream while a run is in progress. Both use server-sent events, and `curl -N` turns off buffering so you see each event as it arrives.

```bash
curl -N "https://app.dozaldevs.com/admin/tenants/$TENANT/tasks/$TASK/status/stream" \
  -H "Authorization: Bearer $KEY"
```

The status stream sends an `event: status` line each time something changes, with `status` and `phase_category` in the data, and ends with `event: done` once the run is terminal. The sibling `.../tasks/:id/progress/stream` sends `event: progress` for each step the run takes. It stops with `event: done` when the run finishes, or after about fifteen minutes if the run is still going. In that case the `done` event says so, and you can open the stream again.

### One run per line

Some Dozals take a list as an input and start one run for each line. The trigger response for those has more in it: `task_ids` for every run that started, `runs` with each line's own `task_id` and `status_url`, and `failures` for any line that didn't start. `task_id` and `status_url` at the top are just the first run's. A list of any length starts a run for every line, and `?dry_run=true` returns a `run_count` so you can check first.

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
- There's no per-Dozal webhook URL to receive your events, and no callback when a run finishes. Your system calls the trigger route and then polls or streams. A Dozal can also start on its own from a schedule, from an event in a connected app, from a tag in Slack, or from another Dozal finishing (see Handoffs below).

### Starting a run from a Slack tag

If the job is "answer when someone asks", don't build a client that polls the trigger route. Let people tag the organization's Slack app in a channel instead. The Dozal starts on its own, and the answer comes back in the same Slack thread.

Turn it on by saving the channel in `trigger_sources` with an Admin key. The channel id starts with `C` (public) or `G` (private), and `base_version` is the Dozal's current version, as with any edit:

```sh
dozaldevs update-dozal "$DOZAL" --data base_version:="$VERSION" \
  --data trigger_sources.type=manual --data trigger_sources.slack_mention.channel_id="$CHANNEL"
```

A Dozal that also runs on a schedule keeps `type: 'scheduled'` and its schedule next to `slack_mention`. To turn the tag start off, save `trigger_sources` again without the `slack_mention` key.

- **The app has to be in the channel.** The save still works when it isn't, but `warnings` says so and the Dozal can't be switched on until that's fixed. For a public channel, `POST /admin/tenants/:tenantId/slack/channels/:channelId/join` adds the app. For a private one, a person has to open the channel and type `/invite @<the app>`, and the join route answers `409` with `PRIVATE_CHANNEL_NEEDS_INVITE`. `GET /admin/tenants/:tenantId/slack/bot-status?channel_id=...` tells you the app's name and whether it's in that channel.
- **Only a real @mention starts a run.** The plain word doesn't, and neither does a reply in a thread that wasn't tagged. A tag in a channel where no Dozal listens gets a short reply saying how to turn it on.
- **Anyone who can post in the channel can start it**, guests and people from other companies in Slack Connect included. Who sits in that channel is the customer's call.
- **The answer lands in the thread.** Anything the run would send or change anywhere else waits for the Dozal's owner in the dashboard, whatever its approval setting says, and only the owner decides it there.
- **Two Dozals in one channel both save, with a warning.** Whoever tags the app is asked which one they mean.
- **A Dozal that listens also needs a report channel.** If it has none, the listen channel becomes its report channel and the answer says so. Clearing it while the tag start is on answers `422` with `REPORT_CHANNEL_REQUIRED`.
- **Some Dozals can't take it.** One that declares a tool with a real-world effect, or, on organizations still on the old engine, one that uses credentials or writes the platform can't hold for the owner, is refused with `422` `SLACK_START_NOT_ALLOWED`. The message says why in plain words and names the apps; `blocker.code` says which rule refused it.

`GET .../archetypes/:archetypeId/schedule` and the create and edit responses carry a `schedule_estimate`: runs per day and the price per day. A schedule that runs more than 24 times a day needs a yes before it saves. Send `confirm_frequent_schedule: true` with the same request once a person has seen the estimate. Without it the answer is `422` `FREQUENT_SCHEDULE_NEEDS_CONFIRM` carrying the `schedule_estimate`. An edit that doesn't touch `trigger_sources` is never refused for this.

## The run's logs

Once a run has started, the API can show you what it recorded. We call the whole set the run's logs. They're all readable at Viewer level, and these are the pieces:

- `GET /admin/tenants/:tenantId/tasks/:taskId/events` is the structured one, and the place to start.
- `GET .../tasks/:taskId/conversation` returns ad-hoc conversations only: the messages a sender wrote and the replies that went back to them. For a Dozal run it's empty (`items` is `[]`), so don't read an empty answer as "nothing happened". What a Dozal was asked and what it sent live in the task's `output`, in its `events` and in its `executions`.
- `GET /admin/tenants/:tenantId/executions?task_id=:taskId` lists each attempt at the run. Add `?id=` for one attempt in full, including its transcript. With the CLI, `list-executions --id "$EXECUTION" --detailed`: the short form leaves the transcript out.
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
- Other apps: the connect route under connections in the spec takes the app's `toolkit` name as a query parameter and returns `{ "url": "..." }`. The `url` is `null` when the app can't be connected that way. It also takes `label` or `connection_id`, explained below.

All three need Member level. Once the person has clicked through, check `list-app-connections` (`GET /admin/tenants/:tenantId/composio/connections`) to see an app show up, or `GET /admin/tenants/:tenantId/integrations` for Slack and GitHub, and `GET .../integrations/attention` for connections that have stopped working. Apps like Gmail or Jira are never in `/integrations`, so don't read their absence there as "not connected".

### Several accounts of one app

An organization can hold several named accounts of one app, like a work Gmail and a job search Gmail. Every account has its own id, and that id is how you point at one. The app's name never is.

- `list-app-connections` returns one item per account. Read `id`, `label`, `toolkit`, `health_status` and `used_by_count` (how many Dozals use that account). Health belongs to the account, so one can fail while another of the same app works.
- The connect route takes `label` to add an account, or `connection_id` to reconnect that exact one. With neither, it reconnects the organization's only account of the app, or adds the first. If the organization has several, it answers `409` with `CONNECTION_CHOICE_REQUIRED`.
- A name is unique within its app and compared without regard to capitals. A taken one answers `409` with `CONNECTION_LABEL_TAKEN`, on connect and on rename alike.
- An existing account that was never fully signed in has to be reconnected before you add another. Adding one first answers `409` with `CONNECTION_NEEDS_RECONNECT`, and the fix is to reconnect that account with its `connection_id`.
- Rename with `PATCH /admin/tenants/:tenantId/composio/connections/:connectionId` and `{ "label": "..." }`.
- Disconnect with `DELETE /admin/tenants/:tenantId/composio/connections/:connectionId`. It removes that account only, and the app's other accounts stay. There's no disconnect by app name.

All three routes need Member level.

A Dozal uses one account per app, and `app_connections` on the Dozal says which. It's an object of `{ "<toolkit>": "<connection id>" }`, sent on create and on PATCH (Admin level, like any Dozal edit). A PATCH merges it key by key, so send only the apps you're changing, and set an app to `null` to remove its choice. An id that isn't an account of that app in this organization is refused with `422` and `INVALID_APP_CONNECTION`. Dozals that already existed when a second account was connected keep using the first, so connecting a second account doesn't change what they do.

With one account there's nothing to choose. With several and no choice made, the Dozal doesn't guess. It waits for a person or a key to pick one, so set `app_connections` for every app a new Dozal uses before you expect it to run.

Ask a Dozal, the organization's built-in assistant, can't be edited like other Dozals. Choose its account with `PATCH /admin/tenants/:tenantId/assistant/app-connections` and `{ "app_connections": { "<toolkit>": "<connection id>" } }`, with the same rules and the same `422`. `GET` on that route reads its choices. Both need Member level.

GitHub has one extra case. If the installation already exists and the person who authorized it can't see it, the callback answers `409` with `GITHUB_LINK_NEEDS_PERSON`. The agent will see GitHub still unconnected in the list. A signed-in person has to link it from Integrations in the dashboard.

## Using your connected apps directly

You don't need a Dozal to use an app the organization already connected. To read or change something in a connected app, run the action directly with `run-app-action` (`run_app_action` over MCP); never create a Dozal just to run one action. Your key can run an action on it, and DozalDevs makes the call with the organization's connection, so you never see the app's password or token.

Not sure which app or action? Ask in plain words first: `search-actions` (`search_actions` over MCP) returns the best matches across every app the organization can use and the platform's own operations.

```bash
dozaldevs search-actions --q "which cleaning jobs are this week"   # best matches, apps and platform alike
dozaldevs list-app-connections                        # what's connected
dozaldevs list-app-actions googlesheets               # its actions (slug, name, kind), and whether your key may run each
dozaldevs get-app-action googlesheets GOOGLESHEETS_BATCH_GET   # one action's description and inputs
dozaldevs run-app-action googlesheets GOOGLESHEETS_BATCH_GET --data arguments.spreadsheet_id=abc --data 'arguments.ranges:=["A1:C10"]'
```

Send one `session_id` of your own with every call that belongs to the same job, such as your conversation id: `--data session_id=<your id>` (saved HTTP endpoints take it too). The organization then sees that job's calls together as one session. Without it, your key's calls group by day (UTC).

The app's inputs go under `arguments`. The answer is `{ "successful": true|false, "data": ..., "error": ..., "provider_status": 200 }`. `successful: false` means the app itself said no (a bad id, a missing sheet), so read `error` and fix the inputs.

Which account the call runs on is `connection_id`, an optional field in the call's body (`--data connection_id=<id>` in the CLI). Leave it out when the organization has one account of the app. With several and none named, the call answers `409` with `CONNECTION_CHOICE_REQUIRED` and runs nothing, so take the id from `list-app-connections` and send it.

What your key may run depends on its level and on the action:

- A Viewer key runs only actions that read.
- Member and above run any action, including the ones that pay or refund money, delete data, cancel something, change an access code or password, remove a person's access, or publish something publicly, and actions DozalDevs hasn't classified yet. Nothing waits for approval and nothing is held, so the human gives a key that level only when they trust it with those actions. The action's kind stays on the call in Activity as `hold_kind`.
- `list-app-actions` shows `allowed_for_you` for each action. For a Viewer key, `null` means the action hasn't been checked yet and is checked the first time someone runs it. The short form of the list has no `description`: to pick an action by what it does, read `get-app-action` for the likely ones, or add `--detailed` to the list for every field. `--detailed` needs dozaldevs 0.5.0 or newer (an older CLI reads the flag as needing a value).

Each call is charged as normal usage and shows on the Billing page as "Direct app calls". Owners and admins see every call, with the key that made it, under "Calls to your apps" on the API Keys page (`list-app-calls`). A call DozalDevs refused (not allowed for your key's level, or the app not connected) is listed too, with `result: "not_run"` and the reason, and is not charged. There are two limits:

- An organization can make a limited number of direct calls a minute. Past it you get `429` with `RATE_LIMITED` and a `Retry-After` header.
- When the platform's monthly app-call allowance runs short, direct calls pause for everyone with `503` and `DIRECT_APP_BUDGET_REACHED`, so Dozals keep working. Try again later.

## Reading a photo or a file

To learn what a photo, scan, PDF, Excel sheet (.xlsx), CSV or text file says, hand it over; no Dozal is needed:

```sh
dozaldevs hand-over-file --data file_name=receipt.jpg --data content_base64="$(base64 < receipt.jpg | tr -d '\n')"
```

`reading.text` is what the file says: every word and number of a photo, a PDF page by page, a sheet as tables. iPhone HEIC photos work. A number two readings of the picture disagree on is marked `[not sure, could be …]`, and something nobody can read is `[unreadable]`; pass those on as they are, never as a guess. When `reading.next` is not null the file is long: read on with `dozaldevs read-file "$FILE_ID" --data from="$NEXT"`. A file up to 20 MB is kept privately for the organization. A 503 `FILE_READER_UNAVAILABLE` kept the file: read it again in a minute with `read-file` and the `file_id` it names.

## Reading a website with its saved sign-in

When the organization saved a sign-in for a website (`GET /admin/tenants/:tenantId/browser-sessions` lists them), `read-browser-site-pages` (`POST /admin/tenants/:tenantId/browser-sessions/sites/:siteKey/read`) with `{"urls": ["https://..."]}` (any number of addresses on that website, or a partner portal the sign-in reaches) opens a browser with that sign-in and returns each page's `address`, `title` and visible `text`, with the read's `read_id`. It is read-only: nothing is clicked, typed or submitted. Each page takes from a few seconds to about a minute and the browser costs a fraction of a cent a minute. A read still going after about 50 seconds answers `202` with its `read_id`; `get-browser-site-read` (`GET /admin/tenants/:tenantId/browser-sessions/reads/:readId?wait_seconds=50`) then returns the pages read so far and `status` (`running`, `read`, `needs_sign_in` or `failed`), so call it again while it says `running`. 409 means a browser is busy (try again in a few minutes) or the sign-in lapsed (the person signs in again); 400 names an address that is not on the website. `run-browser-keep-alive-visit` only checks that the sign-in still works.

## Browsing a website with its saved sign-in

To look, click or type on a website the organization saved a sign-in for, `open-browser-site-session` (`POST /admin/tenants/:tenantId/browser-sessions/sites/:siteKey/browse`, `{"start_url": "https://..."}` on that website) opens a browser signed in and returns a `session_id` and the first `page` (address, title, visible text and its `controls`, each with a `ref`). `take-browse-session-step` (`POST .../browse-sessions/:sessionId/steps`) takes one step: `{"step": {"op": "observe"}}` or `{"op": "goto", "url": ...}` on the same website run at once; `click` (`ref`), `type` (`ref`, `text`) and `select` (`ref`, `value`) answer `needs_confirmation` with `would` (what it would do, in plain words, to show the person) and a `confirm_token`; send `{"confirm_token": ...}` to run it, once, within 10 minutes. A step that reaches a checkout or a payment is refused (403). `end-browse-session` closes the browser; the session also ends by itself after 10 minutes. The saved sign-in is left as it was. The browser's time is billed like a read.

## Signing in with a login saved in 1Password

When the organization connected 1Password, `get-saved-login` (`--data website=shop.example.com`) hands back one website's username, password and current one-time code. It needs a person's own admin or owner key, matches the website's address exactly (never lists the vault), and tells every owner who read which website. Use it to sign in for the person; never store or print the password.

## Charts, diagrams and documents as pictures and files

To turn a chart, a diagram or a document into something you can send anywhere, call `make-card-file` (`make_card_file` over MCP) with the card's `type` (`chart`, `mermaid` for a diagram, or `document`) and its `source`, the same text that goes inside a chat card's fence: chart JSON, Mermaid text, or Markdown that starts with a `# ` title. A chart or diagram becomes a picture (`png`); a document becomes a PDF (`pdf`, the default) or a Word file (`docx`). A document may hold its own charts and diagrams: fence the document with four backticks and each chart with three. The answer is `{ url, file_name, content_type, format, expires_at? }`: a picture's link does not expire, a document's link works for a week. `language` (such as `es`) sets the language of a document's date line.

```bash
dozaldevs make-card-file --data type=chart --data 'source={"type":"bar","title":"Sales","series":[{"name":"2026","data":[{"x":"Jan","y":3}]}]}'
```

The card is checked first, the same way the chat and Dozals check theirs. One that would not draw answers `422` with `CARD_NOT_DRAWABLE` and the reason, so fix it and send it again. The picture is drawn exactly as the chat draws it.

When the assistant answers in a chat with a card, `download-assistant-chat-card` returns that card as a file made from the reply as it was kept: a chart or diagram as `png`, a document as `pdf` or `docx`. Name the chat, the turn (`turn_seq`), the card's position among that turn's cards counting from 1, and the format. A card that can't become that format, or a chat the key's person can't see, answers `404`. Like every chat route it needs a key that stands for a person.

If your own screen could not draw a card (a diagram that won't parse, say), `report-assistant-chat-card-render-failure` tells the chat with the turn, the card's position and the error text; the chat's next turn sees the error, so asking it to fix the card works. One report per card is kept.

## Sharing a page at a public link

`publish-page` puts one whole HTML page (starting with `<!doctype html>`, styles and scripts inline) online at its own link anyone can open with no sign-in, and answers the link as `url`. Sharing the exact same page again answers the same link (`reused: true`) instead of making a second one. `update-page` puts new HTML at an existing page's link; name the page by the `page_id` the first share answered. A page that is not a whole page, or that carries something shaped like a password or key, is refused with `422 PAGE_CHECK_FAILED` and a `message` saying what to fix. Shared pages are listed with the organization's websites as `kind: page`, and taking one down works like taking down a website. A page goes on the open internet, so check with the person before you share it.

## Turning speech into text

`transcribe-dictation` (`POST /admin/tenants/:tenantId/voice-transcriptions`, any level) turns a short recording of someone speaking into text, the same way the dashboard's microphone does when a browser can't listen itself. Send `audio_base64` (the file, base64, up to 3 MiB) and `mime_type` (for example `audio/webm`, `audio/ogg`, `audio/mp4`, `audio/mpeg` or `audio/wav`); it answers `{ "text": "..." }`. Nothing is kept: the audio is not stored or logged. A 422 `DICTATION_NOT_HEARD` means no speech was heard, a 503 `DICTATION_UNAVAILABLE` means try again in a minute. Each call is a small cost recorded for the organization, so send a long recording in pieces of a minute or so rather than one huge file.


## Talking out loud with the chat

The dashboard's talk mode (a spoken conversation with the Ask chat) is three calls, usable with a key for your own voice client:

- `get-live-voice` (`GET /admin/tenants/:tenantId/live-voice`, any level) says whether talk mode is on (`enabled`) and `turn_sound_threshold`, the chance from a sound-based end-of-turn model at or above which a client may treat a turn as finished.
- `hear-live-voice` (`POST /admin/tenants/:tenantId/live-voice/transcriptions`, member and up) takes one piece of speech cut at a pause, as a 16-bit WAV in `audio_base64` (up to 3 MiB), plus `before`, what the person already said in this turn. It answers `{ "text": "...", "finished": true | false | null }`: whether the whole turn sounds finished from its words, or null when it can't tell (then wait a second of quiet). A 422 `LIVE_VOICE_NOT_HEARD` means no speech was heard.
- `speak-live-voice` (`POST /admin/tenants/:tenantId/live-voice/speech`, member and up) takes `text` (one or a few sentences, up to 1,000 characters) and answers the spoken audio as `audio/mpeg`. A 503 `LIVE_VOICE_UNAVAILABLE` means talk mode is off or the voice couldn't be made; try again in a minute.

Send the person's words to the chat as an ordinary message with `spoken: true` so the reply is written to be heard. Nothing is kept: neither the audio nor the text is stored or logged. Each call is a small cost recorded for the organization.
## What still needs a person

An agent can do almost everything, but a few steps are a human's by design:

- Creating the organization. `POST /me/organizations` is person-only, so the first sign-up and the organization it makes happen in a browser.
- The first key. A key can't create another key, so a signed-in owner makes the first one under Setup, API Keys in the dashboard's left menu, picks its level, and copies it. The secret is shown once.
- Consent clicks. Every OAuth connect ends with a person pressing Allow at the other service.
- Remote-browser sign-in. For apps that need a password typed into a real page, `POST /admin/tenants/:tenantId/browser-sessions/start` opens a live browser, and a person signs in there. Airbnb signs in the same way, through the airbnb.com website (the old `/airbnb/login` routes answer 410). An Owner-level key can open these sessions, but it can't type the password.
- GitHub's link-an-existing-installation picker, as above.

Everything else, from hiring and running to approving and watching, goes through the routes in the spec. When you aren't sure whether a route is open to your key, read its `x-auth` first.
