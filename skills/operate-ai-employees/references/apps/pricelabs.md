<!-- Generated from the tools themselves; do not edit by hand. -->

# PriceLabs

The actions your AI can run on PriceLabs through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `pricelabs` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                 | Reads or changes | What it does                                                                                                                |
| ---------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `CHECK_HOSTFULLY_SYNC` | reads            | Check that the nightly prices in Hostfully still match what PriceLabs set, and report every active home that is out of sync |
| `GET_PRICES`           | reads            | Get the nightly prices PriceLabs computed for one listing over a date range                                                 |
| `LIST_LISTINGS`        | reads            | List PriceLabs listings across all channels, optionally narrowed to one channel or to listings with price sync on           |
| `VALIDATE_ENV`         | reads            | Validate that required PriceLabs environment variables are set                                                              |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`check-hostfully-sync` is `CHECK_HOSTFULLY_SYNC`), and `--name` is the input `name`.

### What you can do

- Every PriceLabs tool is read-only. You can't change a price, a listing or a sync setting from here.
- Don't suggest turning price sync on for more listings as a quick fix. PriceLabs bills per listing with sync on, account-wide, so more synced listings cost money.

### Listings and prices

- PriceLabs lists every channel on the account. For Hostfully listings, use `--pms orbirental`: that's PriceLabs' name for Hostfully. The listing `id` is the Hostfully property UID.
- `get-prices` needs `--listing-id`, `--pms`, `--from` and `--to`, all four. Dates are `YYYY-MM-DD`, with `--from` not after `--to`.
- Compare `price` with Hostfully, not `userPrice`. `userPrice` often doesn't match what Hostfully holds. An open night has an empty `bookingStatus` and a `userPrice` other than `-1`.
- `min`, `base` and `max` can be null, and `max` always is on Hostfully listings. Null isn't 0.
- A listing PriceLabs can't price comes back with `error` set (`LISTING_TOGGLE_OFF`, `LISTING_NO_DATA` or `LISTING_NOT_PRESENT`) and no days. That's a normal answer, not a failure.

### The price sync check

- `check-hostfully-sync` takes four flags and all are required: `--days`, `--stale-after-hours`, `--min-difference` and `--timezone`. Use the values your job gives you.
- It reads PriceLabs and Hostfully one call after another, so give the command a long timeout.
- Each active home gets one status. `stale_sync` means the last push is older than your limit. `price_mismatch` means the prices differ. `no_price_days` means PriceLabs returned no prices. `sync_off` and `no_pricelabs_data` mean it couldn't compare.
- `sync_pending` means PriceLabs recalculated after its last push, so a difference is expected until the next push. Count and name these homes in your report. Never fold them into "in sync".
- A home with a per-listing markup differs from Hostfully every day and shows as `price_mismatch` every run. That's real, not a bug in the check.
- `needsAttention` is true when `attention` has any entry. Each entry lists up to 3 example nights, and only the first 15 entries carry examples.

### Limits and errors

- PriceLabs allows 60 requests a minute and 1,000 an hour, then answers 429. Keep reports small and don't loop over every listing for one question.
- `PriceLabs request failed (HTTP 401): API key rejected` (or the same with HTTP 403) means the saved key is wrong. Report it and stop; retrying won't help.
- Never store or repeat a `reservation_id` from a reservation read. It identifies a guest.
