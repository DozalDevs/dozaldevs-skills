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

1. Create your organization in DozalDevs.
2. Under Settings, API keys, create a key at the level the job needs. A viewer key can only read; an admin key can hire and approve.
3. Set `AI_EMPLOYEE_API_KEY` to that key in the shell your assistant runs in.
4. Optionally set `AI_EMPLOYEE_TENANT_ID` if you belong to more than one organization, and `AI_EMPLOYEE_BASE_URL` if you use a different host.

## Try it

Ask your assistant: list my AI employees

## License

MIT. Copyright DozalDevs.
