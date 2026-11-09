# Agent boxes and Dozal organization access

> Reference for the `manage-ai-employees` skill. Load `operate-ai-employees` first.

## Organization access for a Dozal

`set-dozal-org-access` turns on or off whether the runs a person starts of one Dozal may call the organization's own API. It needs **Owner**, and it isn't an assistant tool: an Admin key, a run key or a box key gets `403`.

```sh
dozaldevs set-dozal-org-access "$DOZAL" --data enabled:=true
```

It answers `{ id, org_access_enabled }`, or `404 NOT_FOUND` for an unknown id. Off is the default.

What a flagged Dozal gets: when a person starts a run (Run now) and is still a member, the run gets a key in `DOZALDEVS_API_KEY` at that person's current role, capped at Admin and never Owner. It lasts about 65 minutes and nothing revokes it early. A run no person started (a schedule, a handoff, `run-dozal` from an API key) gets no key. Its `api/` tools then fail with a message saying a person must start it with Run now from a Dozal that has organization access turned on. That message is the answer, not a bug to retry.

## Agent boxes

A box is a hosted browser agent, already logged in to the organization with the `dozaldevs` CLI. It belongs to one person and acts at that person's own level (never above Admin). Needs **Member**. An API key may start one for the member who made it, and the box's level can't be above the key's own.

```sh
dozaldevs start-agent-box
dozaldevs start-agent-box --data level=MEMBER --data ttl_minutes:=240
dozaldevs list-agent-boxes --status running
dozaldevs get-agent-box "$BOX"        # until ready is true
dozaldevs create-agent-box-link "$BOX"
dozaldevs stop-agent-box "$BOX"
dozaldevs stop-agent-boxes
```

`start-agent-box` answers `201` with `id`, `url`, `status`, `level`, `expires_at`, `ready`, `reopened: false` and `machine_price_per_hour_usd`. There is no password.

A person in the dashboard doesn't need any of this: "Open advanced chat" in the sidebar (or in the `⌘K` palette) starts or reopens their box and opens it in a new tab. Point them there first.

The way in is a one-time link. Poll `get-agent-box` until `ready` is `true` (usually within half a minute), then run `create-agent-box-link "$BOX"` and give the person the `url` it answers: it works for one minute, once, and only for the box's owner. Only the owner can make it, from a signed-in session or an operator key they made: another person's box is `404 NOT_FOUND` even for an Admin, a Dozal run's key (and a box key) is `403 FORBIDDEN`, a box that isn't ready yet is `409 AGENT_BOX_NOT_READY` (poll, then ask again), and `503 AGENT_BOX_LINK_UNAVAILABLE` means the platform has no link secret yet, so tell the platform team. The link is a credential: don't log or paste it anywhere but to the person. If the person already has a live box that is running (and not expired, and ready or only just started) or only just starting, it answers `200` with `reopened: true` and the same `id` and `expires_at`: that is the same box, so don't start another; make a link to it. `list-agent-boxes` and `get-agent-box` never return a password or key: they show `id`, `url`, `status` (`starting`, `running`, `stopping`, `stopped`, `failed`), `expires_at`, `owner_user_id`, `level`, `ready`, who created it and `destroyed_at`; the list is newest first.

Rules a start follows:

- Only organizations listed in the platform setting `agent_box_enabled_tenant_ids` may start one. Anyone else gets `403 AGENT_BOX_NOT_ENABLED`; the platform team has to add the organization. It is not open to customer organizations yet; don't promise it to one.
- A box's own key, an in-app assistant key or a snapshot-apply key cannot start one (`403 FORBIDDEN`). A person, their operator key, or a run they started with Run now can.
- One live box per person per organization: starting again reopens it (above). A start that loses a race with another start of the same person is `409 AGENT_BOX_LIMIT`; open the box you have instead. A box that is `failed`, `stopping`, expired, stuck `starting` or never became ready is stopped by the next start, which then makes a new one (`502 AGENT_BOX_STOP_INCOMPLETE` if that stop fails; nothing is created, retry later).
- A reopen never goes above this request's level: with a live `ADMIN` box, an Owner using a member-level key or an Admin asking for `MEMBER` gets `409 AGENT_BOX_LEVEL_MISMATCH`. Stop the box, then start again at the level you need. A lower-level live box is reopened as is, never raised.
- The person is the signed-in user, or for an API key the member who made it. A service token, a platform owner who isn't a member, a key with no creator and a removed member get `403 AGENT_BOX_MEMBER_REQUIRED`: use a person's session or a key a current member made.
- `level` is optional, `MEMBER` or `ADMIN`, and can't be above the person's own level or the key's (`403 FORBIDDEN`). Left out, it is the highest allowed (an Owner gets `ADMIN`).
- `ttl_minutes` is optional, from 15 up to the setting `agent_box_max_hours` times 60 (8 hours unless changed), and defaults to that maximum. Outside that is `400 INVALID_REQUEST`.
- The box needs no model key and no organization secret: its AI is the platform's own, billed to the organization as usage on the Billing page's "Advanced chat" line, and a box's machine is charged by time when it stops. **Say what it costs before you start one**: `machine_price_per_hour_usd` in the start answer (and in `get-agent-box-availability`, which you can read first) is the box's rated price per hour, plus the AI as it is used. Don't start a box for someone without telling them.

Other errors: `503 AGENT_BOX_LINK_UNAVAILABLE` (no link secret on the platform; nothing was created), `409 AGENT_BOX_STOPPED` (someone stopped it while it was starting, start a new one), `502 AGENT_BOX_START_FAILED` (nothing was left running, retry in a few seconds), `503 SERVICE_UNAVAILABLE` (boxes aren't configured on this platform). Every error answer carries a `next_step` saying what to do.

Who sees and stops what: a Member lists, gets and stops only their own boxes (another person's id is `404 NOT_FOUND`; `stop-agent-boxes` stops only theirs). An Admin or above, a platform owner and a service token see every box of the organization and can stop any one or all, but only a person can start one. `get-agent-box` also reads a stopped or failed box, so you can see why it ended.

Stopping revokes the box's key and deletes the box. `stop-agent-box` takes one id (`404 NOT_FOUND` if it isn't live or isn't yours), `stop-agent-boxes` stops every live box you may see, including ones still starting, and answers `204`. `502 AGENT_BOX_STOP_INCOMPLETE` means the key is revoked but the box wasn't fully deleted yet. It's marked `failed` and a sweep removes it within five minutes, so list again later instead of retrying in a loop.

A box closes itself after `agent_box_idle_minutes` (30 unless changed) with nothing happening in it: no request from its tab other than the live-update stream, nothing typed in its terminal, and no work by its agent. Its machine time is charged up to then. A box also ends when its owner loses access: removing the member or giving them a lower role, deactivating the account, and suspending or deleting the organization all stop the owner's (or the organization's) boxes. A reaper also takes down a box stuck `starting` or `stopping`, or `running` but never ready, for more than `agent_box_ready_grace_minutes` (10 unless changed), and a running box whose machine is already gone.

Anyone who gets into the box (only the owner, through the link) acts as the box's level. The box holds no AI key; its model calls go through the platform and are billed to the organization. Say so before you hand one out.
