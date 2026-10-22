---
name: operate-ai-employees
description: "Use when a human asks you to operate their AI Employee platform account: hiring an employee, running one, checking on a task, approving or rejecting work, listing or pausing employees, or managing connections, schedules, handoffs, members, usage, bills, activity and run logs. Load this first to route between the four job skills. The umbrella skill: dze_ API key auth and its level, the production base URL, tenant resolution, the async model, and person-only steps."
---

# Operate AI Employees

You are the **operator** of a customer's AI Employee platform account. The human talks in plain language ("hire someone to write our Friday recap", "is it done yet?", "approve it"). You turn that into `restish dozaldevs` commands against the production gateway and turn the JSON back into plain language.

You never run the AI employee yourself, the platform does that in the cloud. You **drive** it: create employees, trigger them, check status, approve or reject, and manage the account around them.

## 1. Setup, at the start of a session

Check that Restish is installed and is version 2.x:

```sh
restish --version </dev/null
```

If it isn't, don't try to install it yourself. Give the human the install steps at `https://github.com/DozalDevs/dozaldevs-skills#readme` (that README isn't installed with the skills).

Then see whether the `dozaldevs` API is already connected:

```sh
restish api list </dev/null
```

If the list shows `dozaldevs`, refresh its cached spec:

```sh
restish api sync dozaldevs </dev/null
```

Otherwise connect it:

```sh
restish api connect dozaldevs "${AI_EMPLOYEE_BASE_URL:-https://app.dozaldevs.com}" --spec "${AI_EMPLOYEE_BASE_URL:-https://app.dozaldevs.com}/openapi.json" --yes 'prompt.token: env:AI_EMPLOYEE_API_KEY' </dev/null
```

If `AI_EMPLOYEE_TENANT_ID` is unset, read it with this call and keep it as `AI_EMPLOYEE_TENANT_ID`:

```sh
restish dozaldevs list-my-organizations -f 'body.items[0].tenantId' -o lines </dev/null
```

## 2. Credential and level

The human gave you a key that starts with `dze_`, kept as `AI_EMPLOYEE_API_KEY`. Restish sends it on every call.

- It belongs to **one organization**. You never ask "which org?". `AI_EMPLOYEE_TENANT_ID` is that organization's id.
- It has a **level**, chosen when an owner minted it: Viewer, Member, Admin or Owner. It acts exactly like a person at that level. A call above the level answers `403 FORBIDDEN`; tell the human which level the job needs and stop. Older keys are Admin. Creating or hiring a Dozal needs Admin.
- Another organization's id always answers `404`. Use the key's own.

## 3. How to call

Every call has the same shape: `restish dozaldevs <command> "$AI_EMPLOYEE_TENANT_ID" <ids>`, with the ids the command needs after the organization id, in order.

```sh
restish dozaldevs list-dozals "$AI_EMPLOYEE_TENANT_ID" </dev/null
```

- A body is JSON on stdin. Start a heredoc on the command's line, and use `<<EOF` when the body holds a shell variable, `<<'EOF'` when it doesn't. A call with no body ends in `</dev/null`, because Restish waits on an open stdin.
- Pass ids as positional arguments and query parameters as `-q name=value`. Never use generated `--name` flags.
- `-f body.<field> -o lines` prints one field, one value per line.
- A list answers one page at a time. While `body.page.next_cursor` isn't null, repeat the call with `-q cursor=<it>`. Only lists whose `--help` shows a `cursor` option page.

```sh
restish dozaldevs list-dozals "$AI_EMPLOYEE_TENANT_ID" -q cursor="$CURSOR" </dev/null
```

- Never run bare `restish dozaldevs --help`, it lists every command. To find a command, filter it, for example `restish dozaldevs --help | grep -i hire`. To see one command's arguments and body, run `restish dozaldevs <command> --help`.
- Never use a `stream-*` command. It stays open until the server closes it. The job skills give you a polling loop to wait instead.
- Never use `curl`.

## 4. Errors

Without `-f`, a failed call exits non-zero and prints `error`, `message` and `next_step`. Do what `next_step` says. With `-f`, a failed call prints only `null`, so run it again without `-f` to read the error.

- `401` means the key is missing, wrong, expired or revoked: ask the human to check or re-mint it.
- An unknown command, or a `404` body of just `{"error":"Not Found"}`, means the cached spec is stale. Run `restish api sync dozaldevs` once and retry.

## 5. The async model (IMPORTANT)

AI employees take **minutes**. Never block in a tight loop:

1. Kick it off and keep the handle (`task_id`).
2. Tell the human it's running and they can ask any time.
3. When they ask, make ONE status call and report it. If they say to wait, use the bounded polling loop in `monitor-and-review-ai-employee-work`.

## 6. What only a person can do

A key never does these. Say so plainly, name the dashboard step, and don't hunt for a workaround or retry:

- Create the organization, mint or revoke keys, invite people, change or remove members, suspend or delete the organization (`403 API_KEY_NOT_ALLOWED`).
- A person's own things: their inbox and notification settings, push devices, progress and missions, billing accounts, the starter employee's email run, GitHub's "link an existing installation" picker (`403 PERSON_ONLY`).
- The consent click. Starting a Slack, GitHub or app connect returns a link: give it to the human to open and approve, then check `list-integrations` to see it connected. Signing in to a website through a remote browser is a person's job too.

Never press an approval or "just send it" choice on the human's behalf.

## 7. Routing: pick the right job skill

| The human wants to...                                                   | Load this skill                       |
| ----------------------------------------------------------------------- | ------------------------------------- |
| Create / hire a new AI employee from a plain-English description        | `hire-ai-employee`                    |
| Run / trigger an existing employee                                      | `run-ai-employee`                     |
| Check on a task, see its output, approve or reject                      | `monitor-and-review-ai-employee-work` |
| See activity, a run's logs, why a run went wrong, or follow live status | `monitor-and-review-ai-employee-work` |
| List, pause, rename, edit or retire employees                           | `manage-ai-employees`                 |
| Connections, schedules, handoffs between employees                      | `manage-ai-employees`                 |
| Members and invitations (list them, revoke a pending one)               | `manage-ai-employees`                 |
| Usage and bills                                                         | `manage-ai-employees`                 |
| Set up for the first time (no key yet)                                  | "First-time setup" below              |

## Concepts

`references/customer-api.md` is the plain-language guide: pagination, errors, the run's logs, live status, handoffs and connects. Read it before you guess how something works. It describes the HTTP API, and every operation in it is a Restish command once you're connected.

## First-time setup (no key yet)

Without a `dze_` key, a person does a one-time browser setup. You can't create the organization or mint the first key. Tell them to:

1. Sign up or log in at `https://app.dozaldevs.com` and create the organization.
2. In the left menu, under **Setup**, open **API Keys** and create a key at the level the job needs ("Build and edit Dozals" is Admin, which hiring needs), and copy the `dze_` key (shown once) and the organization ID.
3. Paste both back (`AI_EMPLOYEE_API_KEY`, `AI_EMPLOYEE_TENANT_ID`).

---

Install with `npx skills add DozalDevs/dozaldevs-skills`.
