# Checking what an employee is told, and undoing a change

> Reference for the `manage-ai-employees` skill. Load `operate-ai-employees` first.

## See exactly what it's told

Read its compiled instructions before and after a fix, and whenever its output isn't what its steps ask for: two instructions that contradict each other show up here side by side. A declared `output_schema` appears under "The Shape Your Deliverable Must Have", and that shape wins over anything the steps say about format.

```sh
restish dozaldevs preview-dozal-brain "$AI_EMPLOYEE_TENANT_ID" "$DOZAL" -f body.compiled_agents_md -o lines </dev/null
```

## Undo a change

List the employee's edits (newest first) and revert the one you want gone. A revert puts the employee back exactly as it was before that edit, and needs `base_version` like any content edit. If it would also have to change a setting a revert never touches (`vm_size`, `temperature`, `concurrency_limit`, `worker_env`, `pre_check_enabled`, `pre_check_adapter`), it answers `409 REVERT_BLOCKED` with those names in `fields` and nothing changes. Set them back with `update-dozal` first, then revert again.

```sh
restish dozaldevs list-dozal-edits "$AI_EMPLOYEE_TENANT_ID" "$DOZAL" </dev/null
restish dozaldevs revert-dozal-edit "$AI_EMPLOYEE_TENANT_ID" "$DOZAL" "$EDIT" <<EOF
{"base_version":$VERSION}
EOF
```
