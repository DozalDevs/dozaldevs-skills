# Roles

> Reference for the `manage-ai-employees` skill. Load `operate-ai-employees` first.

## Custom roles and role changes

Besides Owner, Admin, Member and Viewer, an organization's owners can make roles of their own. A custom role starts from one of Admin, Member or Viewer and switches jobs on or off; jobs that only an Owner may do (set the organization up, billing, keys for other software, deleting the organization) can never be in one. A custom role never ranks above the role it started from.

```sh
dozaldevs list-custom-roles
dozaldevs list-role-changes
```

`list-custom-roles` shows each role with what it can do and how many people hold it. `list-role-changes` is the record of who made, changed, deleted or gave a role. A key can't make, change or delete a role (`403 API_KEY_NOT_ALLOWED`) or ask an owner for access (`403 PERSON_ONLY`): tell the human to do that in the dashboard, on the Team page's Roles tab.
