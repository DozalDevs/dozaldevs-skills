<!-- Generated from the tools themselves; do not edit by hand. -->

# PostgreSQL

The actions your AI can run on PostgreSQL through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `postgres` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action       | Reads or changes      | What it does                                                                                               |
| ------------ | --------------------- | ---------------------------------------------------------------------------------------------------------- |
| `DELETE_ROW` | changes (delete data) | Delete rows from a Postgres table where a search column equals a value, and return the affected row count. |
| `FIND_ROWS`  | reads                 | Select rows from a Postgres table matching a parameterized WHERE condition and return them.                |
| `INSERT_ROW` | changes               | Insert a single row into a Postgres table and return the inserted row.                                     |
| `RUN_QUERY`  | changes               | Execute a parameterized SQL statement against a Postgres database and return the resulting rows.           |
| `UPDATE_ROW` | changes               | Update rows in a Postgres table where a search column equals a value, and return the affected row count.   |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`delete-row` is `DELETE_ROW`), and `--name` is the input `name`.

### Reading and writing

- Put values in `--args` and refer to them as `$1`, `$2` and so on. Never put a value inside the SQL or the `condition` text. Only the values are bound; the rest is sent as written.
- `find-rows` takes `--table`, a `--condition` such as `name = $1`, `--args` as a JSON array, and optionally `--columns` as a JSON array. Table and column names may hold only letters, digits and `_`.
- `run-query` runs any statement and returns `{rows, rowCount}`. Prefer `find-rows` for a plain read. Use `run-query` for joins and counts.
- `insert-row` returns the stored row. `update-row` and `delete-row` match on one column equal to one value (`--search-column`, `--search-value`) and return `{rowCount}`. Check the count; zero means nothing matched.
- `update-row` and `delete-row` change every row that matches. Be sure the column is specific enough.
- `delete-row` can't be undone and may wait for the owner's approval.
- If the job names a connection, pass it with `--connection <name>`. Without it you get the default one.
