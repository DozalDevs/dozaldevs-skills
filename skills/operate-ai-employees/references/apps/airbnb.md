<!-- Generated from the tools themselves; do not edit by hand. -->

# Airbnb

The actions your AI can run on Airbnb through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `airbnb` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action               | Reads or changes | What it does                                                                                       |
| -------------------- | ---------------- | -------------------------------------------------------------------------------------------------- |
| `CALENDAR`           | reads            | Per-night availability and host price for one Airbnb listing over a date range                     |
| `EARNINGS`           | reads            | Report paid-out and upcoming earnings totals for a date range on the connected Airbnb host account |
| `INSIGHTS`           | reads            | Report Airbnb performance-dashboard metrics (ratings and the like) for the host account            |
| `LISTING_DETAIL`     | reads            | Look up one Airbnb listing's full detail by its listing id                                         |
| `LISTINGS`           | reads            | List every listing on the connected Airbnb host account                                            |
| `MESSAGES`           | reads            | Read the Airbnb host inbox: list threads, or read one thread by id                                 |
| `MESSAGES_LIST`      | reads            | List the threads in the Airbnb host inbox, newest first, with a last-message preview               |
| `MESSAGES_THREAD`    | reads            | Read the most recent messages inside one Airbnb inbox thread                                       |
| `RESERVATION_DETAIL` | reads            | Look up one Airbnb booking's full detail by its confirmation code                                  |
| `RESERVATIONS`       | reads            | List reservations on the connected Airbnb host account (upcoming by default)                       |
| `REVIEW_RESPOND`     | changes          | Post a public host response to a guest review (permanent; delivery-phase, approval-gated)          |
| `REVIEWS`            | reads            | List the reviews guests left on one Airbnb listing                                                 |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`calendar` is `CALENDAR`), and `--name` is the input `name`.

### Reading

- Every read tool takes `--format table|json|csv`, default `table`. A table is for reading, not parsing. Use `--format json` when you need to process the result.
- `listings` lists every listing. `listing-detail --listing-id` gives one listing in full.
- `calendar --listing-id` shows each night's availability and price. Without `--from` and `--to` it covers today through the next 30 days.
- `reservations` lists upcoming reservations. Add `--past` for earlier ones, and `--limit` to cap the count. `reservation-detail --code` takes a confirmation code.
- `messages-list` shows inbox threads, newest first, 100 at most unless you pass `--all`. `messages-thread --thread-id` reads the recent messages in one thread.
- `reviews --listing-id` lists a listing's reviews. `earnings` covers the current calendar year unless you set `--from` and `--to`. `insights` defaults to the `QUALITY` metric group.

### Sending and replying

- `messages-send` and `review-respond` can't be undone. A message reaches the guest at once, and a review response is public and permanent. Use them only in a delivery step, with text that was approved.
- Don't pass a thread or review id on a real run. The tool already knows which thread or review to answer. For long text, write it to a file and use `--text-file` (`messages-send`).
- If a send tool prints `not sent by you; the platform sends the approved reply after this step`, that's the right outcome. It exits 0 on purpose. Don't retry it and don't send the message another way.
- When a tool says the Airbnb connection needs to be reconnected (exit code 42), stop. The saved sign-in no longer works and only the owner can sign in again. Retrying won't help.
