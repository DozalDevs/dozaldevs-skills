<!-- Generated from the tools themselves; do not edit by hand. -->

# DoorLoop (website sign-in)

The actions your AI can run on DoorLoop (website sign-in) through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `doorloop_web` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action    | Reads or changes      | What it does                                                                                                                                 |
| --------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `ACTIONS` | reads                 | List the DoorLoop actions available through the DoorLoop sign-in: name, kind, area, ids, query keys and whether each takes a body. Read-only |
| `CHANGE`  | changes               | Run one DoorLoop create or update action through the DoorLoop sign-in, e.g. createWorkflow, with its ids and JSON body                       |
| `DELETE`  | changes (delete data) | Run one DoorLoop delete action through the DoorLoop sign-in, e.g. deleteWorkflow, with its ids                                               |
| `READ`    | reads                 | Run one DoorLoop read action through the DoorLoop sign-in, e.g. listTasks or getLease, with its ids and query. Read-only                     |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`actions` is `ACTIONS`), and `--name` is the input `name`.

### What you can do

- Start with `doorloop_web/actions`: it lists every action you can call, what kind it is, the ids it needs, its query keys and whether it takes a body.
- Read with `doorloop_web/read --action listTasks --query '{"page_number":1,"page_size":50}'`. Lists come back as `{data, total}`; page with `page_number` until you have `total` rows.
- `doorloop_web/change` creates or updates and `doorloop_web/delete` deletes. Each may wait for the owner's approval. Read the record first, and send DoorLoop's own field names in `--body`.
- Every tool prints one line of JSON: `{action, result}`, `result` being DoorLoop's answer.

### When it fails

- If DoorLoop isn't signed in or the sign-in expired, the run ends blocked. Say a person needs to sign in to DoorLoop again on the Websites page. Don't retry.
- If a change failed with DoorLoop's own message, fix the request from that message. Never send the same change twice without reading DoorLoop first; the first may have gone through.
