# Checking what an employee is told, and undoing a change

> Reference for the `manage-ai-employees` skill. Load `operate-ai-employees` first.

## What a save changed

Every save (`update-dozal`, `apply-hiring-chat`, `revert-dozal-edit`) answers with `changes`: one `{field, before, after, source}` per field whose saved value actually changed. Trust it rather than re-reading the Dozal, and tell the human what it says, not what you asked for. `source: "platform"` means the platform rewrote that field while saving; its `reason` says why and `requested` holds what you sent. An empty `changes` with a `message` means nothing was saved, and `ignored_fields` names keys the platform accepts but never stores (`instructions`: send `execution_steps` instead).

Shortly after a save that changes the steps, the platform rewrites `job_description` to match them, as its own edit (it shows in the edit list as made by the platform). You can't set it directly. That edit moves `version` too, so a save sent with the version from before it is refused: read the Dozal again and resend.

## Check an update before it goes live

Add `--validate_only true` to `update-dozal`. It runs every check and platform rewrite and answers with the Dozal as it would be saved, its `changes` and `validated_only: true`, but saves nothing. Show the human those `changes`, then send the same body without the flag.

## What only a direct update changes

The edit chat changes the steps, sends, notes, name, schedule, where results go, apps, per-run inputs, approval and the output format (`output_schema`). It can't change `department`, `concurrency_limit`, `worker_env`, `work_item_key`, `overview`, `pre_check_enabled`, `pre_check_adapter`, `pre_check_config`, `vm_size`, `appearance`, `estimated_manual_minutes_override`, `temperature`, `platform_rules_override`, `template_auto_update`, `on_hours` or `app_connections` (which account it uses for each app), and says so when asked: use `update-dozal` for those. Sending `null` clears a field that can be empty, e.g. `{"output_schema": null, "base_version": 7}` removes a required output format.

## See exactly what it's told

Read its compiled instructions before and after a fix, and whenever its output isn't what its steps ask for: two instructions that contradict each other show up here side by side. A declared `output_schema` appears under "The Shape Your Deliverable Must Have", and that shape wins over anything the steps say about format.

```sh
dozaldevs preview-dozal-brain "$DOZAL" --field compiled_agents_md
```

## Undo a change

List the employee's edits (newest first) and revert the one you want gone. A revert puts the employee back exactly as it was before that edit, and needs `base_version` like any content edit. If it would also have to change a setting a revert never touches (`vm_size`, `temperature`, `concurrency_limit`, `worker_env`, `pre_check_enabled`, `pre_check_adapter`, `on_hours`), it answers `409 REVERT_BLOCKED` with those names in `fields` and nothing changes. Set them back with `update-dozal` first, then revert again.

```sh
dozaldevs list-dozal-edits "$DOZAL"
dozaldevs revert-dozal-edit "$DOZAL" "$EDIT" --data base_version:="$VERSION"
```
