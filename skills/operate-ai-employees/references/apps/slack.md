<!-- Generated from the tools themselves; do not edit by hand. -->

# Slack

The actions your AI can run on Slack through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `slack` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                | Reads or changes | What it does                                                                                               |
| --------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------- |
| `POST_MESSAGE_INLINE` | changes          | Post a Slack message whose body is supplied directly as a string, for short text                           |
| `READ_CHANNELS`       | reads            | Read messages from one or more Slack channels within a lookback window; thread replies only when asked for |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`post-message-inline` is `POST_MESSAGE_INLINE`), and `--name` is the input `name`.

### Posting

- Pick the tool by the text: `post-message-inline` for short text, `post-message-from-file` (with `--text-file`) for long or multi-line text. `post-message` takes either `--text` or `--text-file`.
- `--channel` is an id such as `C0123456789` or a name such as `#ops`. Write plain Markdown; it's converted for Slack.
- Messages from one task thread together on their own. Don't pass `--thread-ts` unless your instructions give you one. `--no-thread` does nothing.
- `--blocks` takes a JSON array of Slack blocks. You rarely need it.
- If your run was started from a Slack tag, post only in the thread you were asked in. Posting at the top of a channel, or in another thread, is refused.
- A post can't be taken back. Post once. If you post the exact same message again in the same run, it is skipped, and the tool tells you.
- If a send tool prints `not sent by you; the platform sends the approved reply after this step`, that's the right outcome. It exits 0 on purpose. Don't retry it and don't send the message another way.
- If the app isn't in a private channel, the post fails with `slack_not_in_channel`. Report it. Someone has to invite the app. For a public channel, the app joins by itself.

### Reading

- `read-channels --channels "C0123,general,#ops"` reads recent messages. `--lookback-hours` defaults to 24. Replies come only with `--with-replies`.
- Slack lets the app read history about once a minute per workspace, so each channel can cost about a minute, and each thread with replies another. Read only the channels and threads the job needs.
- A channel Slack was still limiting after one wait comes back under `unreadable`. Say so in your report, and don't read it again in the same run. A long history can come back with only its newest messages and an `incomplete` note: say that too, or read a shorter `--lookback-hours`.
- To cover the whole workspace, use `read-channels --channels all`. It reads every channel the app is in, private ones included, and lists under `notMember` the public channels the app isn't in. Those aren't read: say so in your report, because someone has to add the app to them.
- `read-channels --channels all --list-only` lists every channel the app can see, with `isMember`, and reads no messages. Use it to plan which channels to read within Slack's one-read-a-minute limit.
