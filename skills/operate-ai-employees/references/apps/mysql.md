<!-- Generated from the tools themselves; do not edit by hand. -->

# MySQL

The actions your AI can run on MySQL through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `mysql` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action          | Reads or changes      | What it does                                                                                                                                                             |
| --------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `DELETE_ROW`    | changes (delete data) | Delete every row in a MySQL table whose search column equals a given value. The value is bound as a ? parameter.                                                         |
| `EXECUTE_QUERY` | changes               | Run an arbitrary SQL statement against the MySQL database and return its results. Use ? placeholders and pass the values separately to avoid SQL injection.              |
| `FIND_ROWS`     | reads                 | Read rows from a MySQL table matching a SQL WHERE condition, optionally limited to specific columns. Use ? placeholders in the condition and pass the values separately. |
| `GET_TABLES`    | reads                 | List the names of all tables in the connected MySQL database (SHOW TABLES).                                                                                              |
| `INSERT_ROW`    | changes               | Insert a new row into a MySQL table from a map of column names to values. Values are bound as ? parameters.                                                              |
| `UPDATE_ROW`    | changes               | Update every row in a MySQL table whose search column equals a given value, setting the supplied column-to-value pairs. Values are bound as ? parameters.                |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`delete-row` is `DELETE_ROW`), and `--name` is the input `name`.

### Reading and writing

- Put values in `--args` and mark their place with `?`. Never put a value inside the SQL or the `condition` text. Only the values are bound; the rest is sent as written.
- Start with `get-tables` when you don't know what exists. It lists table names.
- `find-rows` takes `--table`, a `--condition` such as `name = ?`, `--args` as a JSON array, and optionally `--columns` as a JSON array. Table and column names may hold only letters, digits and `_`.
- `execute-query` runs any statement and returns `{results}`. Prefer `find-rows` for a plain read. Use `execute-query` for joins and counts.
- `update-row` and `delete-row` match on one column equal to one value and change every row that matches. Be sure the column is specific enough.
- `delete-row` can't be undone and may wait for the owner's approval.
- If the job names a connection, pass it with `--connection <name>`. Without it you get the default one.
