# Saved setups (org snapshots)

> Reference for the `manage-ai-employees` skill. Load `operate-ai-employees` first.

## What a saved setup holds

Each non-system employee with its settings (name, identity, steps, sends, risk model, triggers, tools, inputs, department, status), plus departments, handoffs, rules, and knowledge entries. Only `confirmed` and `rejected` rules are saved. Deleted and `superseded` employees are left out.

Not saved, and never changed by an apply: sites, connections, secrets, custom HTTP endpoints, channels, members, API keys, templates, billing and run history. Also not saved: `on_hours`, the `confirm_*` price confirmations, and where an employee delivers.

## Saving

```sh
dozaldevs save-org-snapshot --data label="$LABEL" \
  --data warmup_runs:='[{"archetype_id":"'"$DOZAL"'","inputs":{}}]' \
  --data preserve_archetype_ids:='["'"$DOZAL"'"]'
```

A label is letters, digits, `.`, `_` and `-`. Saving never overwrites: it adds the label's next `version`. Leave out `warmup_runs` or `preserve_archetype_ids` and they're copied from the label's newest version, so a plain re-save keeps both. A list you send, even an empty one, replaces the old one. If the newest version can't be read, `409 SNAPSHOT_PREVIOUS_UNREADABLE`: send both lists. `delete-org-snapshot` removes one version.

- `warmup_runs`: runs started after the restore (up to 50), each an employee id and its inputs. Listing the same one twice starts it twice.
- `preserve_archetype_ids`: employees (up to 200) an apply leaves entirely alone. Their open runs aren't cancelled and their rules, knowledge and handoffs aren't saved or changed. System employees are always left alone.

## Applying

`apply-latest-org-snapshot --data label=...` uses the label's newest version; `apply-org-snapshot "$ID"` one version. It needs **Admin**. When a Dozal calls it, the delete-data permission decides whether a person must approve first.

An apply cancels every open run (except preserved and system employees), with the cancellation code `org_snapshot_restored`. It deletes employees not in the saved setup, then puts back departments, employees, handoffs, rules and knowledge so they match, then starts the warm-up runs. It doesn't wait for them: `get-org-snapshot-apply` shows each warm-up run's `task_statuses` as they are now, so keep polling for those.

It runs in the background, one change at a time. Poll `get-org-snapshot-apply "$APPLY"` until `status` isn't `running`:

- `succeeded`: everything matches.
- `partial`: something wasn't restored. `dozal_outcomes` lists each employee that was `refused` (a check said no, e.g. an activation gate, or the platform rewrote a field) or `missing` (gone from the database, never re-created), and a warm-up that got no task also counts.
- `failed`: the apply died; `error` says why. The organization isn't locked, so start again.

Applying the same version again is safe: it only changes what still differs.

Errors: `409 APPLY_IN_PROGRESS` (one apply per organization at a time, wait for it), `404 NOT_FOUND` (unknown label, or an id from another organization).
