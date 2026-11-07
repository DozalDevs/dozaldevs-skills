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

A box is a hosted, password-protected browser agent, already logged in to the organization with the `dozaldevs` CLI. It belongs to one person and acts at that person's own level (never above Admin). Needs **Member**. An API key may start one for the member who made it, and the box's level can't be above the key's own.

```sh
dozaldevs start-agent-box
dozaldevs start-agent-box --data level=MEMBER --data ttl_minutes:=240
dozaldevs list-agent-boxes --status running
dozaldevs get-agent-box "$BOX"
dozaldevs stop-agent-box "$BOX"
dozaldevs stop-agent-boxes
```

`start-agent-box` answers `201` with `id`, `url`, `username`, `password`, `status`, `level`, `expires_at`, `ready` and `reopened: false`. The password is shown once and stored nowhere, so give it to the human right away and don't paste it anywhere else. If the person already has a live box that is running (and not expired) or only just starting, it answers `200` with `reopened: true`, the same `id` and `expires_at`, and `password: null`: that is the same box, so don't start another or look for a password. `list-agent-boxes` and `get-agent-box` never return a password or key: they show `id`, `url`, `status` (`starting`, `running`, `stopping`, `stopped`, `failed`), `expires_at`, `owner_user_id`, `level`, `ready`, who created it and `destroyed_at`; the list is newest first.

Rules a start follows:

- Only organizations listed in the platform setting `agent_box_enabled_tenant_ids` may start one. Anyone else gets `403 AGENT_BOX_NOT_ENABLED`; the platform team has to add the organization.
- One live box per person per organization: starting again reopens it (above). A start that loses a race with another start of the same person is `409 AGENT_BOX_LIMIT`; open the box you have instead. A box that is `failed`, `stopping`, expired or stuck `starting` is stopped by the next start, which then makes a new one (`502 AGENT_BOX_STOP_INCOMPLETE` if that stop fails; nothing is created, retry later).
- A reopen never goes above this request's level: with a live `ADMIN` box, an Owner using a member-level key or an Admin asking for `MEMBER` gets `409 AGENT_BOX_LEVEL_MISMATCH`. Stop the box, then start again at the level you need. A lower-level live box is reopened as is, never raised.
- The person is the signed-in user, or for an API key the member who made it. A service token, a platform owner who isn't a member, a key with no creator and a removed member get `403 AGENT_BOX_MEMBER_REQUIRED`: use a person's session or a key a current member made.
- `level` is optional, `MEMBER` or `ADMIN`, and can't be above the person's own level or the key's (`403 FORBIDDEN`). Left out, it is the highest allowed (an Owner gets `ADMIN`).
- `ttl_minutes` is optional, from 15 up to the setting `agent_box_max_hours` times 60 (8 hours unless changed), and defaults to that maximum. Outside that is `400 INVALID_REQUEST`.
- The box's model comes from the organization secrets `AGENT_BOX_MODEL_API_KEY`, `AGENT_BOX_MODEL_BASE_URL` and `AGENT_BOX_MODEL_ID`, never from the request. If one is missing: `409 AGENT_BOX_MODEL_MISSING`. Store the three secrets and start again.

Other errors: `409 AGENT_BOX_STOPPED` (someone stopped it while it was starting, start a new one), `502 AGENT_BOX_START_FAILED` (nothing was left running, retry in a few seconds), `503 SERVICE_UNAVAILABLE` (boxes aren't configured on this platform). Every error answer carries a `next_step` saying what to do.

Who sees and stops what: a Member lists, gets and stops only their own boxes (another person's id is `404 NOT_FOUND`; `stop-agent-boxes` stops only theirs). An Admin or above, a platform owner and a service token see every box of the organization and can stop any one or all, but only a person can start one. `get-agent-box` also reads a stopped or failed box, so you can see why it ended.

Stopping revokes the box's key and deletes the box. `stop-agent-box` takes one id (`404 NOT_FOUND` if it isn't live or isn't yours), `stop-agent-boxes` stops every live box you may see, including ones still starting, and answers `204`. `502 AGENT_BOX_STOP_INCOMPLETE` means the key is revoked but the box wasn't fully deleted yet. It's marked `failed` and a sweep removes it within five minutes, so list again later instead of retrying in a loop.

A box also ends when its owner loses access: removing the member or giving them a lower role, deactivating the account, and suspending or deleting the organization all stop the owner's (or the organization's) boxes. A reaper also takes down a box stuck `starting` or `stopping` for more than `agent_box_ready_grace_minutes` (10 unless changed).

Anyone with the password acts as the box's level, and can read the model key. Say so before you hand one out.
