# DozalDevs skills

Let your coding assistant (Claude Code, Cursor, Codex and others) run your DozalDevs AI employees for you: hire one, run it, check its work, and approve it. You ask in plain words.

Works on macOS, Linux and Windows. You need Node 20 or newer.

## 1. Install

```
npx skills add DozalDevs/dozaldevs-skills
npm install -g dozaldevs
```

## 2. Add your key

At https://app.dozaldevs.com, open **Setup**, then **API Keys**, and create a key. Pick "Build and edit Dozals" to try everything. Then run:

```
dozaldevs login
```

Paste the key when it asks. It's saved on this computer.

In more than one organization? Run `dozaldevs login` again with the other organization's key. Each key is saved as its own profile, named after its organization, and the newest login is the one in use.

```
dozaldevs profiles            # what's saved, and which one is in use
dozaldevs use acme            # switch to acme for every command from now on
dozaldevs whoami              # which organization a command would use right now
dozaldevs list-dozals --profile acme   # one command in acme, without switching
```

Your assistant checks `dozaldevs whoami` before it starts, and uses `--profile` when you name an organization.

## 3. Try it

Ask your assistant: **list my AI employees**

Or run it yourself:

```
dozaldevs list-dozals --human
```

## Good to know

- `dozaldevs help` lists every command. `dozaldevs help run` finds the ones about running.
- `dozaldevs version` says whether the CLI or these skills need an update, and what's new. Update with `npm i -g dozaldevs@latest` and `npx skills update`. Your assistant also tells you when a command says an update is out.
- Can't install globally? Type `npx dozaldevs` wherever this page says `dozaldevs`.
- On a server or in CI, set `DOZALDEVS_API_KEY` to the key instead of logging in.
- Key levels: "See everything and read from your connected apps, change nothing" can only read. "Run Dozals, approve their work, and use your connected apps" can also run employees, approve their work, and run actions on the organization's connected apps. "Build and edit Dozals" can also hire and edit them.
- Calling the API from your own code? It's plain HTTPS at `https://app.dozaldevs.com`, described at `https://app.dozaldevs.com/openapi.json`. Send the key as `Authorization: Bearer dze_...`. The [customer API guide](https://github.com/DozalDevs/dozaldevs-skills/blob/main/skills/operate-ai-employees/references/customer-api.md) has examples.

## License

MIT. Copyright DozalDevs.
