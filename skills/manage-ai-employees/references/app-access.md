# App access

> Reference for the `manage-ai-employees` skill. Load `operate-ai-employees` first.

## Giving an employee an app it needs

An employee's steps can say to use an app (create a Jira issue, add a row to a Google Sheet) that the employee has no tools for, so its runs fail or skip that part. The employee's setup checklist shows this as an `app_access` item naming the `app`:

```sh
dozaldevs get-dozal-setup-checklist "$DOZAL_ID"
```

Tell the human in plain words which app it is and what the employee will be able to do there, and ask before you change anything. Only after they say yes:

```sh
dozaldevs give-dozal-app-access "$DOZAL_ID" --data app="$APP"
```

It adds that app's tools to the employee as an edit in its history, made on the human's behalf, so it can be undone like any other edit. `granted: false` means the employee could already reach the app and nothing changed. `added` lists the tools it got.

When it can't be done yet, the answer says why:

- `APP_ACCESS_NEEDS_CONNECTION`: the app isn't connected in the organization. Connect it first (see "Connections" in this skill), then ask again.
- `APP_ACCESS_NEEDS_RECONNECT`: the app's sign-in stopped working. A person signs in again, then ask again.
- `APP_ACCESS_NO_TOOLS`: there are no tools to add, so change the employee's tools on its page instead.
- `409 VERSION_CONFLICT`: the employee changed meanwhile. Read the checklist again and repeat.
