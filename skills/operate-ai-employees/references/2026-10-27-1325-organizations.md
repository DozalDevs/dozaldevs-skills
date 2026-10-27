# More than one organization

A key belongs to one organization. A person in several organizations saves each organization's key once with `dozaldevs login`, and each becomes a **profile**, named after its organization unless they picked a name with `--profile`. One profile is active, and every command uses it.

- `dozaldevs whoami` says which profile and organization a command would use right now (`profile`, `chosen_by`, `organization.name`). Run it at the start and whenever the human names an organization.
- `dozaldevs profiles` lists the saved ones (`items[].name`, `items[].organization`, `active`). It never shows a key.
- For a command in another organization, add `--profile <name>`, for example `dozaldevs list-dozals --profile acme`. It leaves the active profile alone, so prefer it.
- `dozaldevs use <name>` changes the active profile for every later command, in every terminal. Run it only when the human asks to switch.
- Keep one job in one organization. A task or Dozal id from one organization answers `404` in another, so pass the same `--profile` on every call of that job.
- `DOZALDEVS_PROFILE=<name>` works like `--profile` for a whole shell. Which key wins, first match: `--profile`, `DOZALDEVS_PROFILE`, `DOZALDEVS_API_KEY`, then the active profile.
- A profile name that isn't saved fails with exit code 2 and lists the saved ones. It never falls back to another organization.
- No profile for the organization the human means? A person adds it with `dozaldevs login` and that organization's key. You don't, and you never run `use`, `login` or `logout` on your own.
