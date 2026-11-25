<!-- Generated from the tools themselves; do not edit by hand. -->

# Google Calendar

The actions your AI can run on Google Calendar through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `googlecalendar` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                      | Reads or changes | What it does                                                                                 |
| --------------------------- | ---------------- | -------------------------------------------------------------------------------------------- |
| `CREATE_EVENT`              | changes          | Create a Google Calendar event, with optional guests, repeat rule and video meeting          |
| `EVENTS_GET`                | reads            | Read one Google Calendar event by its id                                                     |
| `EVENTS_LIST`               | reads            | List the events on one Google Calendar, optionally within a time range or matching some text |
| `EVENTS_LIST_ALL_CALENDARS` | reads            | List events across several Google Calendars at once within a time range                      |
| `FIND_EVENT`                | reads            | Find events on one Google Calendar by text, time range or type                               |
| `FIND_FREE_SLOTS`           | reads            | Find the free time between busy periods on one or more Google Calendars                      |
| `FREE_BUSY_QUERY`           | reads            | Check when one or more Google Calendars are busy within a time range                         |
| `GET_CALENDAR`              | reads            | Read one Google Calendar's settings by its id                                                |
| `GET_CURRENT_DATE_TIME`     | reads            | Get the current date and time in a time zone                                                 |
| `GET_CURRENT_USER`          | reads            | Show the signed-in Google Calendar user                                                      |
| `LIST_CALENDARS`            | reads            | List the calendars the signed-in Google account can see                                      |
| `PATCH_EVENT`               | changes          | Change some fields of an existing Google Calendar event and leave the rest as they are       |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`create-event` is `CREATE_EVENT`), and `--name` is the input `name`.

- Every Google Calendar tool works on the calendars of the Google account your organization connected; the calendar id `primary` is the account's own calendar, others come from `list-calendars`.
- To see events use `events-list` (one calendar) or `events-list-all-calendars` (several, with a time range); `find-event` searches by text. `events-get` reads one event by id. Pass times as ISO 8601 and a time zone name such as `America/Chicago`; call `get-current-date-time` first when you need today's date.
- For open times use `find-free-slots` (gaps between busy periods) or `free-busy-query` (busy periods only; its items are JSON such as `[{"id":"primary"}]`).
- `create-event` adds an event (give a start and an end, or a start and a duration; attendees are a JSON list of emails or objects); `patch-event` changes only the fields you pass. Both are writes and guests may be emailed an invitation, so say who will be invited.
- A 403 means the connected person lacks access to that calendar; say so plainly instead of retrying.
