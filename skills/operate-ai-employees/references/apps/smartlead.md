<!-- Generated from the tools themselves; do not edit by hand. -->

# Smartlead

The actions your AI can run on Smartlead through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `smartlead` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                           | Reads or changes   | What it does                                                                                                                                                             |
| -------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ADD_CAMPAIGN_EMAIL_ACCOUNTS`    | changes            | Add sending accounts to a Smartlead campaign and read back its final account list                                                                                        |
| `ADD_CAMPAIGN_LEADS`             | changes            | Add up to 100 leads to a Smartlead campaign and report how many it uploaded and skipped; a running campaign then emails them (irreversible)                              |
| `ADD_LEAD`                       | changes            | Add one lead to a Smartlead campaign with its merge values; the campaign then emails them on its own schedule (irreversible once running)                                |
| `ARCHIVE_CAMPAIGN`               | changes            | Archive a Smartlead campaign so it stops sending, and read its status back                                                                                               |
| `CAMPAIGN_HEALTH`                | reads              | Count one campaign's emailed, bounced and opted-out people and say whether it crosses the stop rules you pass                                                            |
| `CAMPAIGN_STATS_INLINE`          | reads              | Count a Smartlead account's sending in a window, with every input given inline: account-wide totals, and one row per listed campaign, per email step and per A/B variant |
| `CREATE_CAMPAIGN`                | changes            | Create an empty Smartlead campaign by name and return its id                                                                                                             |
| `GET_LEAD`                       | reads              | Look up a Smartlead lead by email: its campaigns, last email sent, last reply, and whether it is unsubscribed                                                            |
| `GET_THREAD`                     | reads              | Read the email thread with one Smartlead lead: what was sent to them and what they replied, oldest first                                                                 |
| `LIST_CAMPAIGN_LEADS`            | reads              | List one campaign's leads with their sequence status, optionally for one status                                                                                          |
| `LIST_CAMPAIGNS`                 | reads              | List the Smartlead campaigns with their id, name, and status                                                                                                             |
| `LIST_EMAIL_ACCOUNTS`            | reads              | List the Smartlead sending accounts with their address, domain, daily limit, warm-up status and reputation                                                               |
| `PAUSE_CAMPAIGN`                 | changes            | Pause one Smartlead campaign and read its status back; it can never start or resume one                                                                                  |
| `PAUSE_LEAD`                     | changes            | Stop a lead's remaining emails in Smartlead (pause the sequence) without unsubscribing them                                                                              |
| `REMOVE_CAMPAIGN_EMAIL_ACCOUNTS` | changes            | Remove sending accounts from a Smartlead campaign and read back its final account list                                                                                   |
| `RESUME_LEAD`                    | changes            | Restart a paused Smartlead lead so its sequence continues                                                                                                                |
| `SET_CAMPAIGN_SCHEDULE`          | changes            | Set a Smartlead campaign's sending days, hours, time zone, gap between emails and daily new-lead cap                                                                     |
| `SET_CAMPAIGN_SEQUENCE`          | changes            | Set a Smartlead campaign's email steps with their delays and optional variants, and read them back                                                                       |
| `SET_CAMPAIGN_SETTINGS`          | changes            | Set a Smartlead campaign's open and click tracking and plain-text sending; a reply always stops the lead                                                                 |
| `START_CAMPAIGN`                 | changes            | Start or resume a Smartlead campaign that has leads, and read its status back; it then emails on its schedule                                                            |
| `UNSUBSCRIBE_LEAD`               | changes (contacts) | Unsubscribe a lead in Smartlead so it is never emailed again by any campaign                                                                                             |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`add-campaign-email-accounts` is `ADD_CAMPAIGN_EMAIL_ACCOUNTS`), and `--name` is the input `name`.

### Looking things up

- `list-campaigns` gives campaign ids, names and status. `list-campaign-leads --campaign-id` lists a campaign's leads with their sequence status (`STARTED`, `INPROGRESS`, `COMPLETED`, `PAUSED`, `STOPPED` or `BLOCKED`); use `--status` to narrow, and `--offset` for the next page.
- `get-lead --email` shows a lead's campaigns, last email sent, last reply and whether they unsubscribed. `get-thread --email` reads the email thread, oldest first. Pass `--campaign-id` only when the lead is in several campaigns.
- `campaign-health --campaign-id` counts a campaign's emailed, bounced and opted-out people and applies stop rules you pass as numbers (`--min-emailed`, `--bounce-pct`, `--optout-pct`, `--optout-count`). Add `--sender-block-count N --time-zone <IANA zone>` to also pause on N sends the mail provider refused (Smartlead's sender-originated bounces) on one calendar day in that zone, whatever the emailed count, and `--since <ISO instant>` to judge only sends from that moment on (a campaign resumed after a fix starts clean). It prints one JSON line with `verdict` (`PAUSE` or `OK`), `reasons` and a ready `line`; copy the line, do no arithmetic. The platform reads that printed line back from the tool call ledger to decide whether a finished guard run is trusted, so keep `campaignId` and `verdict` in it. An opt-out counts only for someone already emailed.
- `campaign-stats` counts sending in a window and writes the result to `--out-file`. It needs `--window-file`, `--campaign-ids` and `--out-file`.

### Acting on leads

- `add-lead` needs `--campaign-id`, `--email`, `--first-name` and `--company-name`. The campaign then emails them on its own schedule, which can't be undone once it runs. If Smartlead refuses the lead, the tool prints `{added: false, reason}` and that's a normal outcome. Report the reason.
- `--custom-fields` is a JSON object of merge field names to text.
- `pause-campaign --campaign-id` pauses a whole campaign and prints the read-back status. It never starts or resumes one.

### Building a campaign

- `create-campaign --name` makes an empty campaign and prints its `campaignId`. Then set its emails with `set-campaign-sequence` (a JSON list of steps, each with subject, body, days after the previous step, and optional variants; an empty subject replies in the same thread), its sending window with `set-campaign-schedule`, and its tracking with `set-campaign-settings` (opens and clicks off and plain text unless you say otherwise; a reply always stops the lead).
- `list-email-accounts` shows the sending accounts with their warm-up status and reputation. Attach them with `add-campaign-email-accounts` and detach with `remove-campaign-email-accounts`; both print the campaign's final account list.
- `add-campaign-leads` adds up to 100 leads per call and reports how many were uploaded and why the rest were skipped. Call it again for the next batch.
- `start-campaign` starts a new campaign or resumes a paused one; it refuses a campaign with no leads. Once started, the campaign emails its leads on its schedule and that can't be undone. `archive-campaign` retires a campaign.
- `pause-lead` stops a lead's remaining emails without unsubscribing them. `resume-lead` restarts a paused lead. Leave `--campaign-id` out to act in every campaign the lead is in, or pass it to act in one.
- `unsubscribe-lead` means the lead is never emailed again by any campaign. It may wait for the owner's approval. Don't use it as a pause.
- `reply-to-thread` answers the lead's latest reply. It can't be undone. Write the reply to a file and pass `--body-file`; Markdown is converted for you.

### When it fails

- A 401 means the saved key was rejected. Report it and stop.
