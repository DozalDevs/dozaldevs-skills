<!-- Generated from the tools themselves; do not edit by hand. -->

# Ninety

The actions your AI can run on Ninety through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `ninety` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                       | Reads or changes      | What it does                                                                                                                  |
| ---------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `ADD_ISSUE_COMMENT`          | changes               | Add a comment to an Issue and get back the full comment list.                                                                 |
| `ADD_TODO_COMMENT`           | changes               | Add a comment to a To-Do and get back the full comment list.                                                                  |
| `CREATE_HEADLINE`            | changes               | Create a Headline or Cascading Message on a Team.                                                                             |
| `CREATE_ISSUE`               | changes               | Create an Issue on a Team.                                                                                                    |
| `CREATE_MILESTONE`           | changes               | Create a Milestone on a Rock.                                                                                                 |
| `CREATE_ROCK`                | changes               | Create a Rock owned by the token's user.                                                                                      |
| `CREATE_TODO`                | changes               | Create a To-Do, personal unless a Team is given.                                                                              |
| `DELETE_ISSUE`               | changes (delete data) | Permanently delete an Issue.                                                                                                  |
| `DELETE_KPI_NOTE`            | changes (delete data) | Delete a Measurable's note for one period.                                                                                    |
| `DELETE_ROCK`                | changes (delete data) | Delete a Rock (Ninety soft-deletes it, so it stops appearing in reads).                                                       |
| `DELETE_SCORE`               | changes (delete data) | Delete a Measurable's score for one period.                                                                                   |
| `DELETE_TODO`                | changes (delete data) | Permanently delete a To-Do.                                                                                                   |
| `GET_ACCOUNTABILITY_CHART`   | reads                 | Get the company's Accountability Chart: every Seat, what it is accountable for and who sits in it.                            |
| `GET_ISSUE`                  | reads                 | Get one Issue by its Id.                                                                                                      |
| `GET_KPI_SCORE`              | reads                 | Get one Measurable's score for the period containing a date.                                                                  |
| `GET_MEETING`                | reads                 | Get one concluded Meeting with its notes and the items it worked.                                                             |
| `GET_MILESTONE`              | reads                 | Get one Milestone by its Id.                                                                                                  |
| `GET_NEXT_MEETING`           | reads                 | Get a Team's next upcoming Meeting with its agenda and roster.                                                                |
| `GET_ROCK`                   | reads                 | Get one Rock by its Id.                                                                                                       |
| `GET_TEAM_SCORECARD`         | reads                 | Get a Team's Scorecard: its Measurables and their scores for a window of periods.                                             |
| `GET_TEAM_VTO`               | reads                 | Get a Team's Vision/Traction Organizer: Core Values, Core Focus and the 1-Year Plan and 3-Year Picture goals.                 |
| `GET_TODO`                   | reads                 | Get one To-Do by its Id.                                                                                                      |
| `GET_USER`                   | reads                 | Get one User by their Id.                                                                                                     |
| `LINK_ISSUE`                 | changes               | Link an Issue to a To-Do, Rock or Milestone.                                                                                  |
| `LINK_ROCK`                  | changes               | Link a Rock to a To-Do, Issue or Milestone.                                                                                   |
| `LINK_TODO`                  | changes               | Link a To-Do to a Rock, Issue or Milestone.                                                                                   |
| `LIST_AVAILABLE_TEAMS`       | reads                 | List the Teams the user may pick when creating or querying work.                                                              |
| `LIST_COMPANY_CONVERSATIONS` | reads                 | List the quarterly or annual Conversations across the whole company, one page at a time. Owners and Admins only.              |
| `LIST_COMPANY_HEADLINES`     | reads                 | List the Headlines or Cascading Messages across every Team in the company, one page at a time. Owners and Admins only.        |
| `LIST_COMPANY_ISSUES`        | reads                 | List the Issues across the whole company, one page at a time by cursor. Owners and Admins only.                               |
| `LIST_COMPANY_KPI_GROUPS`    | reads                 | List the KPI groups across the whole company, one page at a time by cursor. Owners and Admins only.                           |
| `LIST_COMPANY_KPIS`          | reads                 | List the Measurables across the whole company with their recent scores, one page at a time by cursor. Owners and Admins only. |
| `LIST_COMPANY_MEETINGS`      | reads                 | List the company's past Meetings across every Team, one page at a time. Owners and Admins only.                               |
| `LIST_COMPANY_MILESTONES`    | reads                 | List the Milestones across the whole company, one page at a time by cursor. Owners and Admins only.                           |
| `LIST_COMPANY_ROCKS`         | reads                 | List the Rocks across the whole company, one page at a time by cursor. Owners and Admins only.                                |
| `LIST_COMPANY_SEAT_HOLDERS`  | reads                 | List who holds which Seat on every Accountability Chart, one page at a time. Owners and Admins only.                          |
| `LIST_COMPANY_SEATS`         | reads                 | List the Seats on every Accountability Chart with their responsibilities, one page at a time. Owners and Admins only.         |
| `LIST_COMPANY_TODOS`         | reads                 | List the To-Dos across the whole company, one page at a time by cursor. Owners and Admins only.                               |
| `LIST_HEADLINES`             | reads                 | List one Team's Headlines or Cascading Messages, one page at a time.                                                          |
| `LIST_MEETINGS`              | reads                 | List one Team's past Meetings, most recent first, one page at a time.                                                         |
| `LIST_ROCK_MILESTONES`       | reads                 | List all the Milestones of one Rock.                                                                                          |
| `LIST_TEAM_USERS`            | reads                 | List the Users on one Team with their roles.                                                                                  |
| `LIST_TEAMS`                 | reads                 | List the Teams the authenticated user belongs to.                                                                             |
| `LIST_USERS`                 | reads                 | List the Users in the company with their roles and Teams.                                                                     |
| `QUERY_ISSUES`               | reads                 | Query Issues, filtered by Team, interval or search text, one page at a time.                                                  |
| `QUERY_KPIS`                 | reads                 | Query Measurables by Team, owner, cadence or text, without their scores.                                                      |
| `QUERY_ROCKS`                | reads                 | Query Rocks by Team, owner, status, level or text, grouped by Team.                                                           |
| `QUERY_ROCKS_PAGED`          | reads                 | Query Rocks by Team, owner, status, level or text, as one flat page with the total count.                                     |
| `QUERY_TODOS`                | reads                 | Query To-Dos by Team, assignee, status, due date or text.                                                                     |
| `QUERY_TODOS_PAGED`          | reads                 | Query To-Dos by Team, assignee, status, due date or text, as one page with the total count.                                   |
| `SET_KPI_NOTE`               | changes               | Create or overwrite a Measurable's note for one period.                                                                       |
| `SET_SCORE`                  | changes               | Create or overwrite a Measurable's score for one period.                                                                      |
| `UNLINK_ISSUE`               | changes (delete data) | Remove the link between an Issue and a To-Do, Rock or Milestone.                                                              |
| `UNLINK_ROCK`                | changes (delete data) | Remove the link between a Rock and a To-Do, Issue or Milestone.                                                               |
| `UNLINK_TODO`                | changes (delete data) | Remove the link between a To-Do and a Rock, Issue or Milestone.                                                               |
| `UPDATE_ISSUE`               | changes               | Update an Issue; only the fields given change.                                                                                |
| `UPDATE_MILESTONE`           | changes               | Update a Milestone; only the fields given change.                                                                             |
| `UPDATE_PERIOD_SCORE`        | changes               | Update a Measurable's score, note or custom goal for one period; each can also be cleared.                                    |
| `UPDATE_ROCK`                | changes               | Update a Rock; only the fields given change.                                                                                  |
| `UPDATE_TODO`                | changes               | Update a To-Do; only the fields given change.                                                                                 |
| `VALIDATE_ENV`               | reads                 | Validate that required Ninety environment variables are set                                                                   |
| `WRITE_SCORES`               | changes               | Write up to 100 Measurable scores in one call; items succeed or fail independently.                                           |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`add-issue-comment` is `ADD_ISSUE_COMMENT`), and `--name` is the input `name`.

### Who you act as

- Every call runs as the person whose token was saved, with that person's permissions in Ninety. A tool they can't use there answers 403.
- `list-company-*`, Seats, Seat holders and Conversations are for Owners and Admins. Anyone else gets 403, and the message says so. Use the team-level lists instead.
- `create-todo`, `create-issue`, `create-headline` and `update-rock` accept `--user-id` when the job says to assign something to someone else.

### Flags and paging

- Write booleans as strings: `--completed true` or `--completed false`, never a bare flag.
- Repeat an array flag for each value: `--user-ids A --user-ids B`.
- Dates are `YYYY-MM-DD`. A Scorecard period date can be any day inside the period.
- Quote JSON flags (`write-scores --items`, `update-period-score --goal`) as one shell value.
- One call returns one page. Loop yourself and stop when a page comes back short or has no `nextToken`. Cursor tools take `--limit` (1 to 100) and `--next-token`. Offset tools take `--page-index` (from 0) and `--page-size`; `query-todos-paged` takes `--page` (from 1).
- Page size has a floor: `list-headlines` 5 to 200, `list-meetings` and `list-company-meetings` 10 to 100. Smaller values are refused.
- Use `query-rocks-paged`, not `query-rocks`. The unpaged one repeats a Rock under every team it belongs to.

### Reading results

- To-Dos carry `_id`, not `id`. Read `_id` first, then `id`. Many fields that look required can be `null`.
- `create-rock` returns an array. `delete-rock` returns the Rock it soft-deleted, and the Rock stops appearing in reads.
- `get-next-meeting` answers 404 when nothing is scheduled. That's a normal answer.

### Writing scores

- `write-scores` takes 1 to 100 items and can answer success while some items failed. The tool exits 1 with one `Error:` line naming every failed item. Read it, fix those items, and send only them again.
- An item fails if its Measurable is a formula, its period hasn't started, or the same Measurable and period appear twice in one call. In the last case both items are rejected, so send each pair once.
- `set-score` overwrites any score already there.

### Writes and deletes

- A write is sent once and never retried. If it fails on a 429 or a network error, check in Ninety whether it landed before you send it again, or you may create a duplicate.
- Every `delete-*` and `unlink-*` is held for the owner's approval when their rules say so.
- Linking the same two items twice answers 409. The link already exists, so move on.
- Ninety's API has no Milestone delete, no Headline update or delete, and no Measurable create. Anything you create while testing stays there, so don't create test data.

### Errors

- 401 means the saved token is wrong or has expired. Report it and stop; the owner has to generate a new token and reconnect Ninety.
- 429 means too many calls (25 a second). Wait at least a second before the next one.
