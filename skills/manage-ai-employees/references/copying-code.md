# Copying a Dozal's code

`dozaldevs code clone <dozal|app|site> <id> [dir]` copies a Dozal's toolbox, an app or a website to this computer with plain `git`. Needs **Admin**. It calls `create-code-access` for a one-hour credential, waits while the code is checked for secrets, clones, and points the copy at `dozaldevs code credential` so every later `git` command gets a fresh credential on its own.

```sh
dozaldevs code clone dozal "$DOZAL"
dozaldevs create-code-access --data target_kind=dozal --data target_id="$DOZAL"   # the raw call: clone_url, username, token, expires_at
```

Push a change only to a branch named `proposals/<short-name>`; the owner approves it in DozalDevs, and live branches refuse pushes. A `409 CODE_SCAN_PENDING` means the secret check is still running (try again in a minute); `423 CODE_HAS_SECRETS` means it found something that looks like a key, and the owner has to clear it first.

## Following your suggestion

Each push to a `proposals/<short-name>` branch becomes a proposal the owner sees. Give it a title and summary in plain words, then watch its checks:

```sh
dozaldevs list-code-proposals --archetype_id "$DOZAL"       # newest first: id, branch, title, status
dozaldevs update-code-proposal "$PROPOSAL" --data title="Reply in the guest's language" --data summary="..."
dozaldevs get-code-proposal "$PROPOSAL"                      # status and each check's result
```

`status` goes `checking`, then `ready` (the owner can approve it) or `failed` (a check found a problem; push a fix to the same branch). Only the owner approves, tries or undoes a change, in DozalDevs; a key can't.
