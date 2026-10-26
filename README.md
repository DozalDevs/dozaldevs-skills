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

## 3. Try it

Ask your assistant: **list my AI employees**

Or run it yourself:

```
dozaldevs list-dozals --human
```

## Good to know

- `dozaldevs help` lists every command. `dozaldevs help run` finds the ones about running.
- Can't install globally? Type `npx dozaldevs` wherever this page says `dozaldevs`.
- On a server or in CI, set `DOZALDEVS_API_KEY` to the key instead of logging in.
- Key levels: "See everything and read from your connected apps, change nothing" can only read. "Run Dozals, approve their work, and use your connected apps" can also run employees, approve their work, and run actions on the organization's connected apps. "Build and edit Dozals" can also hire and edit them.
- Calling the API from your own code? It's plain HTTPS at `https://app.dozaldevs.com`, described at `https://app.dozaldevs.com/openapi.json`. Send the key as `Authorization: Bearer dze_...`. The [customer API guide](https://github.com/DozalDevs/dozaldevs-skills/blob/main/skills/operate-ai-employees/references/customer-api.md) has examples.

## License

MIT. Copyright DozalDevs.
