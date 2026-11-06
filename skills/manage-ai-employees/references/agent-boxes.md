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

A box is a hosted, password-protected browser agent, already logged in to the organization with the `dozaldevs` CLI. Needs **Admin**. An API key may start one, but the box's level can't be above the key's own.

```sh
dozaldevs start-agent-box --data level=ADMIN --data ttl_minutes:=240
dozaldevs list-agent-boxes --status running
dozaldevs stop-agent-box "$BOX"
dozaldevs stop-agent-boxes
```

`start-agent-box` answers `201` with `id`, `url`, `username`, `password` and `expires_at`. The password is shown once and stored nowhere, so give it to the human right away and don't paste it anywhere else. `list-agent-boxes` never returns a password or key: it lists `id`, `url`, `status` (`starting`, `running`, `stopping`, `stopped`, `failed`), `expires_at`, who created it and `destroyed_at`, newest first.

Rules a start follows:

- Only organizations listed in the platform setting `agent_box_enabled_tenant_ids` may start one. Anyone else gets `403 AGENT_BOX_NOT_ENABLED`; the platform team has to add the organization.
- One live box per organization. A second start is `409 AGENT_BOX_LIMIT`.
- `level` is `MEMBER` or `ADMIN` and can't be above the caller's role (`403 FORBIDDEN`).
- `ttl_minutes` is from 15 up to the setting `agent_box_max_hours` times 60 (8 hours unless changed). Outside that is `400 INVALID_REQUEST`.
- The box's model comes from the organization secrets `AGENT_BOX_MODEL_API_KEY`, `AGENT_BOX_MODEL_BASE_URL` and `AGENT_BOX_MODEL_ID`, never from the request. If one is missing: `409 AGENT_BOX_MODEL_MISSING`. Store the three secrets and start again.

Other errors: `409 AGENT_BOX_STOPPED` (someone stopped it while it was starting, start a new one), `502 AGENT_BOX_START_FAILED` (nothing was left running, retry in a few seconds), `503 SERVICE_UNAVAILABLE` (boxes aren't configured on this platform). Every error answer carries a `next_step` saying what to do.

Stopping revokes the box's key and deletes the box. `stop-agent-box` takes one id (`404 NOT_FOUND` if it isn't live), `stop-agent-boxes` stops every live box of the organization, including ones still starting, and answers `204`. `502 AGENT_BOX_STOP_INCOMPLETE` means the key is revoked but the box wasn't fully deleted yet. It's marked `failed` and a sweep removes it within five minutes, so list again later instead of retrying in a loop.

Anyone with the password acts as the box's level, and can read the model key. Say so before you hand one out.
