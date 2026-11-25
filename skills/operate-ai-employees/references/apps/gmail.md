<!-- Generated from the tools themselves; do not edit by hand. -->

# Gmail

The actions your AI can run on Gmail through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `gmail` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                        | Reads or changes | What it does                                                                           |
| ----------------------------- | ---------------- | -------------------------------------------------------------------------------------- |
| `FETCH_EMAILS`                | reads            | List or search emails in the signed-in Gmail account, with their content               |
| `FETCH_MESSAGE_BY_MESSAGE_ID` | reads            | Read one Gmail message by its id                                                       |
| `GET_PROFILE`                 | reads            | Read the Gmail mailbox profile: its address and how many messages and threads it holds |
| `SEND_EMAIL`                  | changes          | Send an email from the signed-in Gmail account                                         |
| `WHO_AM_I`                    | reads            | Show which Google account is signed in to Gmail                                        |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`fetch-emails` is `FETCH_EMAILS`), and `--name` is the input `name`.

- Every Gmail tool works on the Gmail account your organization connected. `fetch-emails` lists or searches (pass Gmail's own search syntax in the query, such as `from:name@example.com newer_than:2d`; it returns one message unless you set the maximum) and `fetch-message-by-message-id` reads one message you already have the id of.
- `send-email` sends from that account: give the recipient, subject and body (list parameters such as cc are JSON, e.g. `["a@example.com"]`); it is a write and may wait for a person's approval. Sending cannot attach files.
- `who-am-i` and `get-profile` tell you which address is connected.
- Read the next page of a long search with the page token from the previous answer.
- A 403 means the connected person lacks access to that mailbox or message; say so plainly instead of retrying.
