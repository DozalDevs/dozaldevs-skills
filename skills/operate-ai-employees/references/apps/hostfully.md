<!-- Generated from the tools themselves; do not edit by hand. -->

# Hostfully

The actions your AI can run on Hostfully through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `hostfully` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                        | Reads or changes       | What it does                                                                                                                                                                                           |
| ----------------------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `FIND_IN_SENT_MESSAGES`       | reads                  | Tell, for each of several strings, whether any message the business side sent in a Hostfully reservation's conversation contains it, reading the whole conversation and showing none of it             |
| `GET_CHECKOUTS`               | reads                  | Fetch the confirmed checkouts of a date: each row holds the reservation facts, the room and city read by the location config, and the same-day arrivals of its group (the help text lists every field) |
| `GET_DOOR_CODE`               | reads                  | Fetch the door code for a Hostfully property from custom data fields                                                                                                                                   |
| `GET_DOOR_CODES`              | reads                  | Fetch the door codes of several Hostfully properties in one call                                                                                                                                       |
| `GET_MESSAGES`                | reads                  | Fetch guest message threads for a Hostfully property from the unified inbox                                                                                                                            |
| `GET_MESSAGES_AGENCY_WIDE`    | reads                  | Fetch guest message threads across every property in the agency (scope from HOSTFULLY_AGENCY_UID)                                                                                                      |
| `GET_MESSAGES_BY_LEAD`        | reads                  | Fetch the full guest conversation for a single Hostfully lead/reservation                                                                                                                              |
| `GET_MESSAGES_BY_PROPERTY`    | reads                  | Fetch guest message threads for every eligible lead on one Hostfully property                                                                                                                          |
| `GET_PRICING_PERIODS`         | reads                  | Fetch the nightly prices Hostfully holds for a property over a date window of at most 365 days                                                                                                         |
| `GET_PROPERTIES`              | reads                  | List all active properties for a Hostfully agency                                                                                                                                                      |
| `GET_PROPERTY`                | reads                  | Fetch details for a single Hostfully property by UID                                                                                                                                                   |
| `GET_PROPERTY_DESCRIPTIONS`   | reads                  | Read a Hostfully property's listing descriptions, one per language                                                                                                                                     |
| `GET_RESERVATIONS`            | reads                  | Fetch reservations (leads) for a Hostfully property or agency                                                                                                                                          |
| `GET_REVIEWS`                 | reads                  | Fetch guest reviews for a Hostfully property or all properties in an agency                                                                                                                            |
| `UPDATE_DOOR_CODE`            | changes (access code)  | Update the door code for a Hostfully property in custom data fields                                                                                                                                    |
| `UPDATE_PROPERTY_DESCRIPTION` | changes (publish site) | Change fields of a Hostfully property's listing description in one language                                                                                                                            |
| `VALIDATE_ENV`                | reads                  | Validate that required Hostfully environment variables are set                                                                                                                                         |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`find-in-sent-messages` is `FIND_IN_SENT_MESSAGES`), and `--name` is the input `name`.

### Ids

- `lead_uid` is the guest's reservation. `thread_uid` is the conversation. They are never the same value. If both come out identical, one is wrong; go back to the trigger payload and read them again.
- `get-messages` takes `--lead-id` with the `lead_uid`, never the `thread_uid`. Pass exactly one of `--lead-id` or `--property-id`. For a sweep of the whole agency, use `get-messages-agency-wide`.
- `--unresponded-only` does nothing when you pass `--lead-id`: you get the whole conversation for that lead.
- `post-guest-approval` needs both `--lead-uid` and `--thread-uid`, as two different values. It has no `--channel` flag; the destination is set for you.

### Reading messages

- Messages come back oldest first. A thread is waiting for a reply when the last message's sender isn't `host` (the output marks it `unresponded`). When the last message is from the host, there's nothing to answer.
- Include `INQUIRY` and `BOOKING_REQUEST` leads, not just `BOOKING`: Airbnb often shows real stays as `INQUIRY`. Leave out `BLOCK` leads, which are calendar blocks.
- An `INQUIRY` lead can come back with no property. Pass `--fallback-property-uid` with the `property_uid` from the trigger.

### Sending

- `send-message` can't be undone. The guest gets it at once on their booking channel. Use it only in a delivery step, after the reply was approved.
- Don't type a lead or thread id into `send-message` on a real run. The platform already knows which guest to answer and refuses a value you made up. If it refuses because it has no recipient, say so in your report.
- Pass long or multi-line text with `--message-file`, not `--message`.

### Prices

- `get-pricing-periods` needs `--property-id`, `--from` and `--to`, all real dates, with `--to` at most 364 days after `--from`. One call returns every night, with no paging.
- A failure reads `Hostfully request failed (HTTP <status>)`. A 401 means the saved key was refused. Report that and stop; retrying won't help.

### Listing descriptions

- `get-property-descriptions --property-id <uid> [--locale en_US]` reads a property's listing text, one entry per language: `name`, `shortSummary`, `summary`, `notes`, `access`, `transit`, `interaction`, `neighbourhood`, `space`, `houseManual` (null when empty).
- `update-property-description --property-id <uid> --locale <locale>` plus one flag per field to change (`--summary`, `--space`, `--notes`, `--house-manual`, ...). Fields you don't pass stay exactly as they are. Text goes as written, emojis included.
- Limits: `name` and `shortSummary` up to 255 characters, `summary` up to 500. Longer text is refused before anything is written, so shorten it and try again.
- The update changes what guests read on the listing, so the owner approves it first by default. Read the current text first and show the owner the before and after.
