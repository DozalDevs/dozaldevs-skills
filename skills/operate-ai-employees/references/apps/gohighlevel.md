<!-- Generated from the tools themselves; do not edit by hand. -->

# GoHighLevel

The actions your AI can run on GoHighLevel through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `gohighlevel` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                 | Reads or changes      | What it does                                                                                                                                                                                      |
| ---------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CHANGE_CONTACT_TAGS`  | changes               | Add and remove tags on a GoHighLevel contact, changing only the tags named (never rewriting the whole list)                                                                                       |
| `CREATE_APPOINTMENT`   | changes               | Book a GoHighLevel contact onto a calendar as a new appointment                                                                                                                                   |
| `CREATE_CONTACT`       | changes               | Create a new contact in GoHighLevel (needs a name, a business name, an email or a phone number)                                                                                                   |
| `CREATE_NOTE`          | changes               | Add a text note to a GoHighLevel contact                                                                                                                                                          |
| `CREATE_OPPORTUNITY`   | changes               | Create a GoHighLevel opportunity for a contact in an existing pipeline stage                                                                                                                      |
| `CREATE_TASK`          | changes               | Create a new task on a GoHighLevel contact                                                                                                                                                        |
| `DELETE_CONTACT`       | changes (contacts)    | Permanently delete a contact from GoHighLevel (irreversible)                                                                                                                                      |
| `DELETE_OPPORTUNITY`   | changes (delete data) | Permanently delete a GoHighLevel opportunity (irreversible)                                                                                                                                       |
| `GET_CONTACT`          | reads                 | Fetch a single GoHighLevel contact by ID                                                                                                                                                          |
| `GET_MESSAGES`         | reads                 | Fetch the message history of one GoHighLevel conversation                                                                                                                                         |
| `GET_OPPORTUNITY`      | reads                 | Fetch a single GoHighLevel opportunity by ID, including its custom field values by name                                                                                                           |
| `GET_PIPELINES`        | reads                 | List GoHighLevel opportunity pipelines and their stages. Read-only — GoHighLevel has no API to create or edit pipelines/stages; a target pipeline/stage must already exist in the GoHighLevel UI. |
| `LIST_APPOINTMENTS`    | reads                 | List the appointments booked for one GoHighLevel contact                                                                                                                                          |
| `LIST_CALENDARS`       | reads                 | List the booking calendars in the configured GoHighLevel location                                                                                                                                 |
| `LIST_NOTES`           | reads                 | List all notes attached to a GoHighLevel contact                                                                                                                                                  |
| `LIST_TASKS`           | reads                 | List all tasks attached to a GoHighLevel contact                                                                                                                                                  |
| `LIST_USERS`           | reads                 | List the team members of the configured GoHighLevel location                                                                                                                                      |
| `SEARCH_CONTACTS`      | reads                 | Search GoHighLevel contacts by name, business name, email, phone, or tag                                                                                                                          |
| `SEARCH_CONVERSATIONS` | reads                 | Search GoHighLevel conversations, optionally filtered by contact or read status                                                                                                                   |
| `SEARCH_OPPORTUNITIES` | reads                 | Search GoHighLevel opportunities by pipeline, stage, contact, status, or free text; each result carries its custom field values by name                                                           |
| `SEND_MESSAGE`         | changes               | Send a message to a contact via GoHighLevel conversations (irreversible)                                                                                                                          |
| `UPDATE_CONTACT`       | changes               | Update an existing GoHighLevel contact (tags REPLACE the whole existing tag list, they are never merged)                                                                                          |
| `UPDATE_OPPORTUNITY`   | changes               | Update an opportunity's fields, including moving it to a different pipeline stage.                                                                                                                |
| `UPDATE_TASK`          | changes               | Update a task on a GoHighLevel contact, including marking it completed                                                                                                                            |
| `UPSERT_BUSINESS`      | changes               | Find a GoHighLevel company by name (case-insensitive) and update it, or create it if none exists                                                                                                  |
| `UPSERT_CONTACT`       | changes               | Create a GoHighLevel contact, or update the existing one matching its email or phone                                                                                                              |
| `VALIDATE_ENV`         | reads                 | Validate that required GoHighLevel environment variables are set                                                                                                                                  |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`change-contact-tags` is `CHANGE_CONTACT_TAGS`), and `--name` is the input `name`.

### Scope

- Every call acts on one GoHighLevel location, the one the connection was made for. You can't list or switch to another.
- A few result shapes are less proven than the rest, mainly `search-contacts` and `search-conversations`. If one comes back empty or odd, read the raw output before you conclude there's nothing there.

### Contacts

- A new contact needs an email or a phone number. `upsert-contact` finds a match by email or phone and updates it, or creates one.
- `--tags` on `update-contact` or `upsert-contact` replaces the whole tag list. Send every tag the contact should keep. To add or remove only some, use `change-contact-tags` with `--add` and `--remove`.
- To link a contact to a company, use `update-contact --business-id`. `create-contact` and `upsert-contact` reject it. `--company-name` is plain text and works on all three. Find or create the company with `upsert-business`.
- `delete-contact` can't be undone and also removes the contact's notes, tasks, conversations and opportunity links. Don't use it unless the job says to.

### Opportunities, tasks and appointments

- Pipelines and stages are read-only. `get-pipelines` lists them, and a pipeline or stage you need must already exist.
- Move a deal by setting `--pipeline-stage-id` with `update-opportunity`. Close one without losing it with `update-opportunity --status lost` (or `won`, `abandoned`). `delete-opportunity` can't be undone.
- `search-opportunities` defaults `--status` to `all`.
- To finish a task, run `update-task --completed true`. There's no separate complete-task tool. `update-task` needs `--contact-id`, `--task-id` and at least one field to change.
- `list-appointments` lists one contact's appointments only. There's no calendar-wide list.
- `create-appointment` needs `--calendar-id`, `--contact-id` and `--start-time`. Write times with a UTC offset, such as `2026-10-01T09:00:00-05:00`, never a trailing `Z`. An end time is only sent if you give one.

### Sending messages

- `send-message` can't be undone. Use it only in a delivery step, after the message was approved.
- `--contact-id` is enough; you don't need a conversation id. `--type` is `SMS` by default, or `Email` or `WhatsApp`. For an `Email`, the text is the body and `--subject` is the subject line. Always set `--subject`, because the default is a generic one.
- For long or multi-line text, write it to a file and use `send-message-from-file --text-file`.

### When a call fails

- A 401 means the saved token was rejected. A 403 means the token lacks a scope the call needs. Report either one and stop; retrying won't help. The owner has to fix the token.
- A 404 on a contact or opportunity means that id doesn't exist in this location. Check the id before you try another.
