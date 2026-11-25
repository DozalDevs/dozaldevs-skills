<!-- Generated from the tools themselves; do not edit by hand. -->

# Google Sheets

The actions your AI can run on Google Sheets through DozalDevs. They are the same tools a Dozal uses, run the same way. Run one with `run_app_action` (MCP) or the run-app-action command, app `googlesheets` and the action's slug below; `get_app_action` shows its inputs. A Viewer key runs only the actions that read.

| Action                        | Reads or changes | What it does                                                                  |
| ----------------------------- | ---------------- | ----------------------------------------------------------------------------- |
| `BATCH_GET`                   | reads            | Read the values of several ranges of one Google Sheet at once                 |
| `FORMAT_CELL`                 | changes          | Format cells of a Google Sheet: bold, colors, number format, borders and more |
| `GET_SHEET_NAMES`             | reads            | List the names of the tabs of a Google Sheet                                  |
| `GET_SPREADSHEET_INFO`        | reads            | Read a Google Sheet's tabs and properties                                     |
| `LOOKUP_SPREADSHEET_ROW`      | reads            | Find the first row of a Google Sheet that contains some text                  |
| `SEARCH_SPREADSHEETS`         | reads            | Search for Google Sheets spreadsheets by name, date or owner                  |
| `UPDATE_DIMENSION_PROPERTIES` | changes          | Change the size or visibility of rows or columns in a Google Sheet            |
| `UPDATE_SHEET_PROPERTIES`     | changes          | Change a tab's properties in a Google Sheet, such as its title or frozen rows |
| `VALUES_GET`                  | reads            | Read the values of a range in a Google Sheet                                  |
| `VALUES_UPDATE`               | changes          | Write values into a range of a Google Sheet                                   |

## Notes

Written for Dozals, so an action is named by its slug in lowercase with dashes (`batch-get` is `BATCH_GET`), and `--name` is the input `name`.

- Every Google Sheets tool works on the spreadsheets of the Google account your organization connected. Spreadsheets are identified by id: find it with `search-spreadsheets` when you only have a name, and `get-sheet-names` for its tab names. Search only finds spreadsheets created through DozalDevs, so when a spreadsheet you expect is not found, ask for its link instead of concluding it does not exist.
- To read: `values-get` (one range), `batch-get` (several ranges at once), `lookup-spreadsheet-row` (the first row containing some text), `get-spreadsheet-info` (tabs and properties).
- To write: `values-update` (the values are a JSON list of rows, e.g. `[["a","b"],["c","d"]]`, and a value input option such as `USER_ENTERED`), `format-cell`, `update-dimension-properties` (row and column size or visibility) and `update-sheet-properties` (a tab's title, frozen rows and so on). These are writes and may wait for a person's approval.
- A 403 means the connected person lacks access to that spreadsheet; say so plainly instead of retrying.
