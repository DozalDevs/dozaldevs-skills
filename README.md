# DozalDevs skills

Skills that let your coding assistant run your DozalDevs account for you: hire an employee, run it, check on its work, approve or reject what it produced, and manage the employees you already have. Your assistant does the work through the DozalDevs command line interface, so you describe what you want in plain words.

There are five skills. Install them together, because four of them link to the guide that ships with the operate skill.

## Install the skills

```
npx skills add DozalDevs/dozaldevs-skills
```

Add `-g` to install them for every project instead of the current one. Run `npx skills update` to pick up new versions.

## Install Restish

The skills call the API through Restish, version 2.3.0. On macOS and Linux:

```sh
mkdir -p "$HOME/.local/bin" && curl -fsSL "https://github.com/rest-sh/restish/releases/download/v2.3.0/restish-2.3.0-$(uname -s | tr '[:upper:]' '[:lower:]')-$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/').tar.gz" | tar -xz -C "$HOME/.local/bin" restish
```

Make sure $HOME/.local/bin is on your PATH. On macOS you can run `brew install restish` instead. On Windows, download the zip from the Restish 2.3.0 release page on GitHub.

## One-time setup

You do this part yourself, because it needs you signed in:

1. Sign up at https://app.dozaldevs.com and create your organization.
2. In the left menu, under Setup, open API Keys and create a key. The screen describes each level in plain words:
   - "See everything, change nothing" is a Viewer key. It can only read.
   - "Run Dozals and approve their work" is a Member key. It can run employees and approve or reject what they produce.
   - "Build and edit Dozals" is an Admin key. It can also hire and edit employees. Pick this one to try everything below.
   - "Everything an owner can, except keys, members, deleting the org and billing" is an Owner key.
3. Copy the key (it starts with `dze_` and is shown once) and set `AI_EMPLOYEE_API_KEY` to it in the shell your assistant runs in.
4. Optionally set `AI_EMPLOYEE_TENANT_ID` to the organization ID shown next to the key. The skills look it up for you when it's unset. Set `AI_EMPLOYEE_BASE_URL` only if you were given a different host.

## Try it

Ask your assistant: list my AI employees

## Use the API without an assistant

Everything the skills do is a plain HTTPS call, so you can also call it from your own code:

- Base URL: `https://app.dozaldevs.com`
- Spec: `https://app.dozaldevs.com/openapi.json` (OpenAPI, no key needed to read it)
- Auth: send the key as `Authorization: Bearer dze_...`
- Guide: [the customer API guide](https://github.com/DozalDevs/dozaldevs-skills/blob/main/skills/operate-ai-employees/references/customer-api.md), shipped with the operate skill, covers keys and levels, lists, errors, runs, approvals, logs, hiring and handoffs, with `curl` examples.

To start an employee from your own system (a cron job, CI, another service), call `POST /admin/tenants/{tenantId}/employees/{slug}/trigger` with a Member key or higher and keep the `task_id` it returns. The guide's "Starting a run from your own system" section has the details, including what isn't there yet.

## License

MIT. Copyright DozalDevs.
