<!-- Generated from the tools themselves; do not edit by hand. -->

# QuickBooks (website sign-in)

The actions your AI can run on QuickBooks (website sign-in) through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `quickbooks_web` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action         | Reads or changes | What it does                                                                                                                                                                                                                         |
| -------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `LIST_HISTORY` | reads            | Count, per bank-feed description, the categories QuickBooks already posted its accepted "For Review" items to. Counting only, no recommendation. Read-only; reads through the platform, which holds the QuickBooks sign-in           |
| `LIST_PENDING` | reads            | List the QuickBooks "For Review" bank-feed items eligible for a category proposal (new-expense adds only), with how many of the rest were skipped and why. Read-only; reads through the platform, which holds the QuickBooks sign-in |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`list-history` is `LIST_HISTORY`), and `--name` is the input `name`.

### What you can do

- You can read QuickBooks bank-feed items. You can't change anything in QuickBooks with these tools.
- `list-pending` returns the items waiting for review that could get a category proposal: new expenses only. It also says how many it left out and why (`not_an_add`, `transfer`, `credit_card_payment`, `not_an_expense`, `linked_transaction`, `match_suggestion`). Report that count with your result.
- `list-history` counts, for each bank description, the categories QuickBooks already posted its accepted items to. Use it as evidence for a proposal. It doesn't pick a category for you; the choice is yours.
- Leave out `--account-id` to read every connected account. Repeat `--account-id` to read only some.
- Both tools print one line of JSON. Read it as JSON.

### When it fails

- If QuickBooks isn't signed in, or the saved sign-in has expired, the run ends blocked. Say that a person needs to sign in to QuickBooks again on the Websites page. Don't retry.
- A refusal that says it is a platform problem isn't fixed by signing in again. Report it as it is.
