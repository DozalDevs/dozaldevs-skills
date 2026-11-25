<!-- Generated from the tools themselves; do not edit by hand. -->

# Google Tasks

The actions your AI can run on Google Tasks through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `googletasks` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action             | Reads or changes | What it does                                                                                 |
| ------------------ | ---------------- | -------------------------------------------------------------------------------------------- |
| `CREATE_TASK`      | changes          | Add a task to a Google Tasks list                                                            |
| `CREATE_TASK_LIST` | changes          | Create a new Google Tasks list                                                               |
| `DELETE_TASK`      | changes          | Delete a task from a Google Tasks list                                                       |
| `GET_TASK`         | reads            | Read one Google Tasks task by its id                                                         |
| `LIST_TASK_LISTS`  | reads            | List the task lists of the signed-in Google account                                          |
| `LIST_TASKS`       | reads            | List the tasks on one Google Tasks list, optionally by due window, completion or update time |
| `UPDATE_TASK`      | changes          | Change some fields of a Google Tasks task and leave the rest as they are                     |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`create-task` is `CREATE_TASK`), and `--name` is the input `name`.

- Every Google Tasks tool works on the lists of the Google account your organization connected, the same lists the person sees in the Tasks app on their phone, in Gmail and in Calendar. Leave `--tasklist-id` out to use their main list; other lists come from `list-task-lists`.
- Use `list-tasks` to see what is on a list and `get-task` for one task by id. Ids come from `list-tasks` or from the answer of `create-task`; never invent one.
- A simple daily note is `create-task` with a title, optional notes and a due date (`--due 2026-11-16`). Google keeps only the date, never a time of day.
- To finish a task use `update-task` with `--status completed`; to reopen it use `--status needsAction`. Only the fields you pass change. `delete-task` removes a task for good, so prefer completing it unless the person asked for removal.
- Finished tasks do not show in `list-tasks` by default: pass `--show-hidden` (and `--show-completed`) to see what was done, or `--due-min` and `--due-max` for a date window. When a page has `nextPageToken`, pass it as `--page-token` for the next one.
- `create-task-list` makes a new list (for example one per project). All four writes change the person's own phone-synced lists, so say what you will add or change.
- A 403 means the connected person lacks access to that list; say so plainly instead of retrying.
