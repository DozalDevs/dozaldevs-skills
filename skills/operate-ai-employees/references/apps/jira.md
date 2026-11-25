<!-- Generated from the tools themselves; do not edit by hand. -->

# Jira

The actions your AI can run on Jira through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `jira` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                  | Reads or changes      | What it does                                                                                      |
| ----------------------- | --------------------- | ------------------------------------------------------------------------------------------------- |
| `ADD_COMMENT`           | changes               | Add a comment to a Jira issue                                                                     |
| `ASSIGN_ISSUE`          | changes               | Assign a Jira issue to a person, or leave it unassigned                                           |
| `CREATE_ISSUE`          | changes               | Create a Jira issue in a project                                                                  |
| `DELETE_ISSUE`          | changes (delete data) | Delete a Jira issue permanently                                                                   |
| `EDIT_ISSUE`            | changes               | Change a Jira issue's fields: summary, description, priority, labels, due date or any other field |
| `FIND_USERS`            | reads                 | Find Jira users by name or email, to get their account id                                         |
| `GET_CREATE_FIELDS`     | reads                 | List the fields of a Jira issue type in a project, and which ones are required to create it       |
| `GET_CURRENT_USER`      | reads                 | Read the Jira account the connected sign-in belongs to                                            |
| `GET_ISSUE`             | reads                 | Read one Jira issue by its key or id                                                              |
| `GET_PROJECT`           | reads                 | Read one Jira project by its key or id, with its issue types and lead                             |
| `LIST_ASSIGNABLE_USERS` | reads                 | List the people who can be assigned issues in a Jira project or on one issue                      |
| `LIST_BOARDS`           | reads                 | List Jira boards, optionally for one project or by name                                           |
| `LIST_COMMENTS`         | reads                 | List a Jira issue's comments, with their text and authors                                         |
| `LIST_FIELDS`           | reads                 | List every Jira field, system and custom, with its id                                             |
| `LIST_ISSUE_TYPES`      | reads                 | List the issue types you can create in a Jira project                                             |
| `LIST_PROJECTS`         | reads                 | List Jira projects, optionally filtered by name or key                                            |
| `LIST_SITES`            | reads                 | List the Jira sites the connected Jira sign-in can reach                                          |
| `LIST_TRANSITIONS`      | reads                 | List the status changes a Jira issue can make right now                                           |
| `SEARCH_ISSUES`         | reads                 | Search Jira issues with a JQL query, one page at a time                                           |
| `TRANSITION_ISSUE`      | changes               | Move a Jira issue to another status (for example To Do, In Progress or Done)                      |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`add-comment` is `ADD_COMMENT`), and `--name` is the input `name`.

- Every Jira tool works on the Jira sign-in your organization connected. If the sign-in reaches more than one Jira site, the tool stops and lists them: pass `--site` with the site's address (e.g. `your-team.atlassian.net`).
- To find issues use `search-issues` with JQL, and always restrict it (a project, an assignee, or a date such as `updated >= -7d`); Jira refuses an unbounded query. Read the next page by passing `--next-page-token` from the previous answer until `isLast` is true.
- Before creating an issue, check the project's issue types (`list-issue-types`) and, when the project has required fields, `get-create-fields`; pass extra or custom fields with `--fields-json` in Jira's own shape.
- To change status use `transition-issue` (it accepts the transition's name or the target status name); `edit-issue` cannot change status, and `assign-issue` changes the assignee.
- People are identified by account id, never by name or email: get it with `find-users` or `list-assignable-users`.
- Descriptions and comments are plain text in and out (`descriptionText` on `get-issue`, `text` on `list-comments`).
- A 403 means the signed-in person lacks that Jira permission; say so plainly instead of retrying.
